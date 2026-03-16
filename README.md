<h1>Windows Event Logging and Security Monitoring</h1>

<img width="800" height="1024" alt="azure event logging and security" src="https://github.com/user-attachments/assets/de8599fa-b637-4a7d-ba32-47068bc57d2d" />

<h2>Project Summary</h2>
<p>
In this lab, I explored how Windows records authentication and security-related activity through Event Viewer. 
The objective was to generate failed logon attempts, trigger an account lockout through Group Policy settings, 
and then review the resulting Security logs to verify that the domain controller enforced the lockout policy correctly.
</p>

<p>
This lab demonstrates how administrators can monitor login activity, investigate failed authentication attempts, 
and confirm that security policies are functioning as intended in a Windows domain environment.
</p>

<p>System administrators and security analysts use Windows Security logs to detect
brute-force attacks, compromised accounts, and unauthorized authentication attempts.</p>

<p>Monitoring failed login events (4625) and account lockouts (4740) allows organizations
to quickly identify suspicious activity and respond to potential security incidents.</p>

<h2>Prerequisites</h2>
<p>
This lab builds on the environment created in the <a href="https://github.com/chrisdegutis/ad-domain-deployment">previous project</a> where an on-premises Active Directory environment was deployed in Microsoft Azure.
Before beginning this lab, the following infrastructure must already be configured:
</p>

<ul>
  <li>A Microsoft Azure subscription</li>
  <li>A Virtual Network configured in Azure</li>
  <li>A Windows Server 2022 virtual machine configured as a <b>Domain Controller</b></li>
  <li>Active Directory Domain Services (AD DS) installed and configured</li>
  <li>A domain created (example: <b>mydomain.com</b>)</li>
  <li>Organizational Units created for <b>_EMPLOYEES</b> and <b>_ADMINS</b></li>
  <li>Domain user accounts created within Active Directory</li>
  <li>A Windows 10/11 client machine (<b>Client-1</b>) joined to the domain</li>
  <li>Remote Desktop access configured for both the Domain Controller and Client machine</li>
  <li>Group Policy configured with an <b>Account Lockout Policy</b> to lock an account after multiple failed login attempts</li>
</ul>

<p>
This environment allows authentication events to be generated and recorded within Windows Security logs, 
which will be analyzed during this lab.
</p>

<h2>Technologies and Environments Used</h2>
<ul>
  <li>Microsoft Azure</li>
  <li>Active Directory Domain Services (AD DS)</li>
  <li>Group Policy Management</li>
  <li>Windows Event Viewer</li>
  <li>Remote Desktop Protocol (RDP)</li>
  <li>Windows Security Logs</li>
</ul>

<h2>Operating Systems Used</h2>
<ul>
  <li>Windows Server 2025 Datacenter (Domain Controller)</li>
  <li>Windows 10 Enterprise (Client-1)</li>
</ul>

<h2>High-Level Steps</h2>
<ol>
  <li>Sign in to the client machine using a domain account.</li>
  <li>Attempt multiple failed logins with an incorrect password.</li>
  <li>Trigger the account lockout policy after the configured threshold is reached.</li>
  <li>Open Event Viewer and review Security logs.</li>
  <li>Identify failed logon events, account lockout events, and successful administrative authentication events.</li>
  <li>Verify that the domain controller enforced the Group Policy lockout settings correctly.</li>
</ol>

<hr>

<h2>Step 1: Sign in to Client-1</h2>
<p>
Begin by connecting to the <b>Client-1</b> virtual machine. This machine is joined to the domain and will be used 
to generate authentication activity for review in Event Viewer.
</p>

<p>
Sign in using a domain account with a domain user (jbaker) so you can later access Event Viewer and review the Security logs.
</p>

<p><b>Example login:</b> <code>mydomain.com\jbaker</code></p>
<hr>

<h2>Step 2: Generate Failed Logon Attempts</h2>
<p>
To create authentication events in the Security log, intentionally attempt to log in to a domain user account using an incorrect password.
</p>

<p>
Enter the wrong password multiple times for the test user account until the account reaches the lockout threshold configured in Group Policy.
</p>

<p>
This step is important because each failed login attempt generates a Security event that can later be reviewed in Event Viewer.
</p>
<img width="800" height="674" alt="image" src="https://github.com/user-attachments/assets/42b44ec9-48e7-4777-9cad-63a8f5f821bd" />
<hr>

<h2>Step 3: Trigger the Account Lockout Policy</h2>
<p>
After the configured number of incorrect password attempts is reached, the domain controller should automatically lock the account.
</p>

<p>
This confirms that the <a href="https://github.com/chrisdegutis/group-policy-adminstration">account lockout policy</a> is being enforced properly through Group Policy.
</p>

<p>
At this point, the locked-out user should no longer be able to authenticate until the lockout duration expires or the account is manually unlocked by an administrator.
</p>
<img width="800" height="1336" alt="image" src="https://github.com/user-attachments/assets/58a3ec94-363d-4067-887d-e0708f9c335f" />

<hr>

<h2>Step 4: Sign in as an Administrator</h2>
<p>
After triggering the lockout, sign back in to <b>Client-1</b> with an administrative domain account so you can review the logs.
</p>

<p>
Use an account such as <b>mydomain.com\jane_admin</b> and password <b>Cyberlab123!</b> to access administrative tools and verify the authentication activity.
</p>

<hr>


<h2>Step 5: Review Authentication Logs</h2>
<p>
Review the authentication events generated during the login attempts. These logs record failed login attempts, 
successful authentications, and account lockout events.
</p>

<p>
On <b>Client-1</b>, open <b>Event Viewer as an administrator</b> and authenticate using 
<b>mydomain.com\jane_admin</b>.
</p>

<p>
Navigate to: <b>Windows Logs &rarr; Security</b>
</p>

<p>
In the Security log, locate the failed logon attempts generated by the incorrect password entries.
</p>

<p>
<b>Note the timestamp of the 5th failed password attempt</b> or whatever threshold was configured in Group Policy.
</p>
<img width="800" height="1338" alt="image" src="https://github.com/user-attachments/assets/583eb235-76bf-4b33-93df-10828a0ec5de" />


<p>
You should also see an <b>Audit Success</b> event indicating that the domain controller successfully enforced the account lockout policy.
</p>

<p><b>Observation:</b> The timestamp of the final failed attempt and the lockout event should align, confirming that the domain controller applied the policy as expected.</p>
<img width="800" height="1122" alt="image" src="https://github.com/user-attachments/assets/f131cd70-cc01-4ac3-9aa0-dbae6eb0c144" />

<hr>

<h2>Step 6: Identify Relevant Event IDs</h2>
<p>
Within the Security log, identify the important event types related to the authentication activity generated in this lab.
</p>

<ul>
  <li><b>4625</b> – Failed logon attempt</li>
  <li><b>4624</b> – Successful logon</li>
  <li><b>4740</b> – A user account was locked out</li>
</ul>

<p>
These Event IDs help administrators determine what occurred, when it occurred, and whether security controls are functioning correctly.
</p>

<hr>

<h2>Step 7: Verify Group Policy Enforcement</h2>
<p>
Compare the failed login attempts and the lockout event to verify that the account lockout policy was enforced according to the configured Group Policy settings.
</p>

<p>
If the lockout occurred after the exact number of failed attempts configured in policy, then the domain controller successfully applied the security settings.
</p>

<p>
This validation is important because it proves that domain-level policies are actively protecting user accounts from repeated unauthorized login attempts.
</p>

<hr>


<h2>Conclusion</h2>
<p>
In this lab, I used Windows Event Viewer to review authentication-related activity in a domain environment. 
By intentionally generating failed logon attempts and triggering an account lockout, I was able to observe how Windows records 
security events and how the domain controller enforces Group Policy settings.
</p>

<p>
This lab explains the use of Windows Security logs, account lockout policies, and how administrators can use 
Event Viewer to investigate authentication issues and verify domain security controls.
</p>
