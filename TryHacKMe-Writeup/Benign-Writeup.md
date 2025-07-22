This is a write-up for the room **[Benign](https://tryhackme.com/room/benign)**
<br>
<br>

<!-- #region -->
<details>
<summary><strong> Task 1: Introduction</strong></summary>
<br>
To start, we first need to click on "Start Machine". After a few minutes, an IP address will be displayed, which must be used for further work. Go to your virtual machine (VM) and open the website 

    🔗 http://YOUR-IP-FROM-HACKTHEBOX 
    
<br>

![Step 1](https://storage.googleapis.com/github-storage-daonware/Benign/step-1.png)
<br>

Next, click on **"Search & Reporting"**

</details>

<!-- #endregion -->

<!-- #region -->
<details>
<summary><strong>Scenario: Identify and Investigate an Infected Host</strong></summary>
<br>
Before we start, we need to process the information we received from Tryhackme. At a later stange, various departments will still be important. <br>
Let´s take a look at the information! <br> <br>

**IT Department**

    - James
    - Moin
    - Katrina 
  
**HR Deppartment**

    - Harron
    - Chris
    - Diana 

**Marketing department**

    - Bell
    - Amelia
    - Deepak
  
</details>

<!-- #endregion -->

<!-- #region -->
<details>

<summary><strong>Question 1: How many logs are ingested from the month of March, 2022?</strong></summary>

<br>

To answer this question, we should first take a look at the dashboard to find out what information it provides us with and how we can interpret this.

<br>

![Step 1](https://storage.googleapis.com/github-storage-daonware/Benign/step-1.png)  

<br>

The images show the dashboard that you see when you first open the app. To continue using the dashboard, we need to clicking on **"Search & Reporting"**

<br>

To answere the questin, we first need to enter a time period after clicking on **"Search & Reporting"**. To do this, click on the search icon (the magnifying glass in the green frame) 
on the right-hand side and then click on **Last 24 hours**. You will see a menu with preset options where you can filter as needed. <br>

The question referred to a specific time period, which could be in the past, present, or future. To change this, we click on **Date Range** below and set the date correctly.

![Step 2](https://storage.googleapis.com/github-storage-daonware/Benign/step-2.png)

![Step 3](https://storage.googleapis.com/github-storage-daonware/Benign/step-3.png) 

<br>

Once you have set the time correctly, let´s see if we can find something to look at and then move on together. If you now enter <code>*</code> in the search bar, the number of events 
will be displayed. 

<br>

⚠️ It should be noted that although the question referred to Windows events, the result is identical if the analysis is limited exclusively to Windows events. For this reason, <code>*</code> was used.

![Step 4](https://storage.googleapis.com/github-storage-daonware/Benign/step-4.png) 


</details>

<!-- #endregion -->

<!-- #region -->
<details>

<summary><strong>Question 2: Imposter Alert: There seems to be an imposter account observed in the logs, what is the name of that user?</strong></summary>

<br>

Before we start check **Log Analysis Using Splunk**

<!-- #region -->

<details><summary><strong>Log Analysis Using Splunk</summary></strong>

<br>

During the post-exploitation phase, we used Splunk Enterprise to analyze Windows Event Logs and identify potential indicator of compromise. By leveraging **Search Processing Language (SPL)**, 
we were able to filter and correlate relevant log events effectively. <br>
Some of the key SPL filters and search terms we use include:

- <code>index=</code>: Specifies the data index to search in e.g., <code>index=wineventlog</code>
- <code>sourcetype=</code>: Defines the type of logs, such as <code>WinEventLog:Security</code>
- <code>EventCode=</code>: Filters specific Windows event IDs
  - <code>4624</code>: Sucessful logon
  - <code>4625</code>: Failed logon attempt
  - <code>4688</code>: New process creation
- <code>Account_Name</code> / <code>User=</code>: Filters by specific usernames
- <code>New_Process_Name=</code>: Helps identify suspicious process executions (e.g., <code>powershell.exe</code>, <code>mimikatz.exe</code>)
- <code>| table</code>: Displays selected field in a structured format
- <code>| stats</code>, <code>| top</code>: Used for statistical analysis and highlighting frequent patterns
- <code>| where</code>: Allows advanced filtering (e.g., <code>| where Account_Name!="SYSTEM"</code>)

**Example Search:**

    index=wineventlog sourcetype="WinEventLog:Security" EventCode=4624 
    | where Account_Name="administator" AND Logon_Type=10
    | table _time, Account_Name, Logon_Type, Workstation_Name

This query allowed us to detect a successful **remote desktop (RPD) logon** to the system using the <code>administrator</code> account. The <code>Logon_Type=10</code> indicates that the login 
occurred via RDP, which can be a sign of unauthorized remote access. <br>
Splunk´s powerfull filtering and correlation features made it easier to track attacker activity and understand how the system was compromised.

</details>

<!-- #endregion -->

<br>

Answering this question requires applying specific filters in the Splunk search bar. One of the first things we can do is specify the index we´re working with:
    
    index="win_eventlogs"

Everyone has their own approach to filtering data, but since we´re specifically looking fpr a **username**, it makes sense to clean up the output and make it more readable. To 
do that, I used the following command:
    
    index="win_eventlogs"
    | dedup UserName
    | table UserName

🔍 Explanation of Filters Used:
- <code>dedup UserName</code>:
  - This command removes **duplicate values** for the specified field - in this case, <code>UserName</code>. If a username appears multiple times in the logs <code>dedup</code> ensures 
  that it only appears **once** in the result. This helps reduce clutter and allows us to focus on unique entries.
- <code>table UserName</code>
  - This formats the output into a clean, simple table showing only the <code>UserName</code> field. Since we´re only interested in usernames, this helps us avoid unrelated log details 
  and makes it easier to scan the results.

![Step 5](https://storage.googleapis.com/github-storage-daonware/Benign/step-5.png)

<br>

At this point, you´ll get a list of all unique usernames found in the log data. The next step is to **compare this list with the list of employees** provide or known. 
If you look carefully, once of the names doesn´t quite belong - and that´s your answer.

<br>

🔍 The name is cleverly hidden, so don´t rush - check closely!

</details>

<!-- #endregion -->

<!-- #region -->

<details>
<summary><strong>Question 3: Which user from the HR department was observed to be running schedule tasks?</strong></summary>
<br>

This task is relatively straightforward - once you know what to look for. After some research, we learn that scheduled tasks are typically executed via a 
file called <code>Schtask.exe</code>. 

<br>

So, we beging by adjusting our search. First, clear any previous input from the search bar, and then enter the following:

    index="win_eventlogs" AND "Schtask.exe"

This search will return a lot of results. ⚠️ But be careful: some of these entries might be from **authorized IT personnel**, and the question specifically asks for a user who 
**shouldn´t** be running schedule tasks - someone from the **HR department**.

<br>

At this point we could refine the search further, but **Splunk actually makes it easier** to filter this visually. <br>

Here´s how:
- On the **left-hand panel** under the **Selected Fields** section, click on the **All Fields** button. <br> ![Step 6](https://storage.googleapis.com/github-storage-daonware/Benign/step-6.png)
- In the search bar that appears, type <code>user</code>, and you´ll see a field named <code>UserName</code>. ![Step 7](https://storage.googleapis.com/github-storage-daonware/Benign/step-7.png)
- Check the box next to <code>UserName</code>, then close the popup.

Now, with the search

    index="wind_eventlogs" AND "Schtask.exe"

still active you´ll see a **field summary** for <code>UserName</code> on the left side. Click on it to view a breakdown of all users who have run this proccess.

![Step 8](https://storage.googleapis.com/github-storage-daonware/Benign/step-8.png) <br>

From there, compare the username against the known HR personnel, and you´ll be able to identify the **unauthorized user from the HR department** who executed the schedule task. <br><br>

    🕵️‍♂️ Hint: The name is in there - you just need to match it carefully. Once of them definitely stands out!

</details>

<!-- #endregion -->

<!-- #region -->

<details>
<summary><strong>Question 4, 5, 6, 7:  Which user from the HR department executed a system process (LOLBIN) to download a payload from a file-sharing host? Also, identify the system binary used, the date it was executed, and the third-party site accessed to retrieve the malicious payload.</strong></summary>
<br>

To solve this task, we first need to understand **what a LOLBIN is**. <br><br>

💡 **What is a LOLBIN?** <br>

**LOLBIN** stand for **Living-Off-the-Land Binary**. These are legitimate, built-in system executables (often found on Windows systems) that can be abused by attacker to perfom malicious actions - such as downloading payloads, executing scripts, or evading detection - **without dropping custom malware** onto the target machine.

Because these binaries are trusted by the operating system and are usually signed by Microsoft, they **bypass most security tools**, making them ideal tools for stealthy attacks. A well-known and reliable reference for LOLBING is the official 
[LOLBAS Project](https://lolbas-project.github.io/#), which documents all known system binaries that can be minused. <br>

![Step 9](https://storage.googleapis.com/github-storage-daonware/Benign/step-9.png) 

<br>

Now that we understand what is and how it can be used, we need to **identify the specific binary used in this case**. To do that, we use Splunk to search for any known LOLBINs by their executable name. After reviewing the list from the LOLBAS project, we apply a filtered search like the following:

    index="win_eventlogs" and "PROGRAMM_NAME.exe"
    | table UserName

Notes: Replace <code>PROGRAMM_NAME.exee</code> with the name of a real LOLBIN from the LOLBAS project, such as <code>curls.exe</code>....  <br>

Once we´ve identified the correct LOLBIN, we can refine the search furtherto extract more details and **answere all four questions at once:**

    index="win_eventlogs" AND "PROGRAMM_NAME.exe"
    | table _time, UserName, CommandLine

- <code>_time</code> will give us the exact date the binary was executed (answering the date question).
- <code>UserName</code> tells us **who** ran the process- which HR user was involved.
- <code>CommandLine</code> reveals **how** the binary was executed, including andy **URLs** or file-sharing hosts accessed to retrieve the malicious payload.

From there, it´s just matter of reviewing the output and identifying: 

1. The **user from HR** who ran the command,
2. The **LOLBIN (system binary)** used,
3. The **execution date** in <code>YYYY-MM-DD</code> format,
4. And the **third-party site** (e.g., a pastebin, Dropbox, or public file host) that was used to download the payload.

</details>

<!-- #endregion -->

<!-- #region -->

<details>
<summary><strong>Question 8: The suspicious file downloaded from the C2 server contained malicious content with the pattern THM{..........}; what is that pattern?</strong></summary>
<br>

To identify the malicious content hidden within the suspicious file/page retrieved from the Command and Control (C2) server, we need to analyze the file/page in a controlled enviroment. Since we´re unsure what exactly the file/page might do (e.g., run malware, inititate outbound connections, or perform system modifications), it´s crucial to open in a sandboxed or virtualized enviroment. Here are two recommende tools for this purpose: <br>

🔍 Option 1: [Browserling](https://www.browserling.com/) <br>

Browserling allows you to open URLs within an isolated browser instance running in the cloud. It´s quick and doesn´t require registation, making it ideal for basic link analysis and checking for obvious indicators like redirects or visible flags. <br>

🔬 Option 2: [Any.Run](https://app.any.run/) <br>

Any.Run is a powerful interactive malware sandbox that lets you upload suspicious files or enter URLs for dynamic analysis. For our use case you can:

1. Go to the dashboard
2. Select **Check Suspicious Links**
3. Paste the URL associated with the C2 Server

    ⚠️ Note: To access the "Check Suspicious Links" feature, you may need to sign in or create a free account.

🏁 **Retrieving the Flag:** <br>

Once the link is submitted, the sandbox will analyze the behavior of the payload. If the payload includes andy visible output - such as a redirect or embedded content - the flag is usually displayed in form:

    THM{example_flag_content}

![Step 10](https://storage.googleapis.com/github-storage-daonware/Benign/step-10.png)

![Step 11](https://storage.googleapis.com/github-storage-daonware/Benign/step-11.png)

📎**Additional Note:**
The **URL you used to retrieve the malicious payload** is also the answer to the previous question: "**What is the URL that the infected host connected to?**"

<!-- #endregion -->



<br><br>

<br>
© 2025 by daonware 
<br>Created: July 23, 2025
<br>Last updated: July 23, 2025

License: [Public Domain / CC0](https://creativecommons.org/publicdomain/zero/1.0/)