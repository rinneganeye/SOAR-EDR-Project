# Automating Incident Response with LimaCharlie (EDR), Tines (SOAR), and Slack
In this project, I combined LimaCharlie (EDR), Tines (SOAR), Slack, and a Windows Server 2022 virtual environment to simulate and automate an incident response workflow. Below, you’ll find the detailed steps to complete this project, along with the tools and techniques used to achieve seamless automation.

# Workflow Summary and Diagram
1. Threat Detection: LimaCharlie detects suspicious activity, such as telemetry from LaZagne.
2. Alert Notification: Tines receives alerts and sends notifications to Slack and email.
3. User Decision: The analyst is prompted to decide whether to isolate the machine.
4. Automated Isolation: Selecting "Yes" triggers Tines to isolate the machine via LimaCharlie API.
5. Confirmation: Slack receives a message confirming the successful isolation.

![Untitled Diagram](https://github.com/user-attachments/assets/2e74615f-5410-46d8-83a9-31a28f087500)

# Tools and Descriptions  

| **Tool**                | **Description**                                                                 |
|--------------------------|---------------------------------------------------------------------------------|
| **VirtualBox**           | An open-source virtualization platform for running multiple operating systems. |
| **Windows Server 2022 Desktop Experience** | A GUI-enabled version of Windows Server 2022 for enhanced usability.          |
| **Slack**| Communication platform for real-time notifications and decision prompts.   |
| **LaZagne Password Tool**| A post-exploitation tool for retrieving stored passwords on a target system.   |
| **LimaCharlie**          | A robust EDR (Endpoint Detection and Response) platform for modern security needs.|
| **Tines**                | A no-code SOAR (Security Orchestration, Automation, and Response) platform.    |

# Steps to Complete the Project
**Step 1: Set Up the Virtual Environment**
1. Download and install VirtualBox on your machine.
2. Create a new virtual machine and install Windows Server 2022 Desktop Experience.
3. Ensure the VM is connected to the internet and configured to allow software installations.

![image](https://github.com/user-attachments/assets/45645972-e1fb-4447-a138-6fdc71369d76)

**Step 2: Deploy LimaCharlie Agent**
1. Sign up for a LimaCharlie account and create a new organization.
2. Download the LimaCharlie agent from the console.
![image](https://github.com/user-attachments/assets/008ce0f8-dfd7-4205-a490-bc8a58943f44)

4. Install the agent on the Windows Server by running the installer and following the instructions.
![image](https://github.com/user-attachments/assets/d573a395-b9bf-41dc-8779-93a85627e1d6)

5. Verify that the agent is active in the LimaCharlie dashboard and can send telemetry data.
![image](https://github.com/user-attachments/assets/d1f0fd27-822a-4245-bc37-c10c6f9f5f01)

**Step 3: Simulate Malicious Activity**
1. Download the LaZagne tool on the Windows Server. (LaZagne is an open-source password-recovery tool that simulates malicious behavior.)
2. Link: https://github.com/AlessandroZ/LaZagne/releases/tag/v2.4.6 (Make sure you disable Windows Real-Time Protection from Windows Virus & Threat Protection settings.) 
3. Run the tool on the server to mimic a password-stealing attack.
![image](https://github.com/user-attachments/assets/93eb35d8-7cbe-4285-8c37-23fb48c5cad0)

4. Verify that LimaCharlie detects the suspicious activity and generates an alert.
5. To check alerts" Go to Sensors > Sensors List > [Your Project Name] > Timeline > Search "lazagne"
![image](https://github.com/user-attachments/assets/0f3bd9dd-58e4-4042-90a6-667e6e06db2f)

**Step 4: Slack Setup**
1. Create a dedicated Slack channel (e.g., #alerts) to receive notifications.
2. Note the Channel ID of the Slack channel, which can be found in the channel details.
3. Prepare to connect Slack to Tines for workflow automation
![image](https://github.com/user-attachments/assets/154265a8-a31d-4b2d-be31-c151cb1a7c61)

# Tines workflow
![Retrieves detection details](https://github.com/user-attachments/assets/313cd968-ed6f-4624-8156-4c80ae2b1de4)

**Step 5: Create a Tines Webhook**
1. In Tines, create a Webhook Action to retrieve alerts from LimaCharlie.
2. Configure the webhook to listen for events and pass relevant alert data to Tines for processing.
3. Test the webhook to ensure it receives events from LimaCharlie.

**Step 6: Add Slack to Tines Workflow**
1. In Slack, navigate to More > Automations, then search for Tines.
2. Add Tines as an integration and set up credentials for the Slack API.
3. In Tines, use the Send a Message action to post notifications to the Slack channel. You can paste the below block into "Message" field. It will send the details to the slack channel. 
```
Title: 
<<retrieves_detection.body.cat>>

Time: 
<<retrieves_detection.body.routing.event_time>>

Computer Name: 
<<retrieves_detection.body.detect.routing.hostname>>

File Path: 
<<retrieves_detection.body.detect.event.FILE_PATH>>

Username: 
<<retrieves_detection.body.detect.event.USER_NAME>>

Command Line: 
<<retrieves_detection.body.detect.event.COMMAND_LINE>>

Sensor ID: 
<<retrieves_detection.body.detect.routing.sid>>

Link: 
<<retrieves_detection.body.link>>
```
4. Input the Channel ID into the Tines workflow for Slack connectivity.
![image](https://github.com/user-attachments/assets/2510c564-21a7-4c8c-a695-af606b67280e)

**Step 6: Configure Email Notifications**
1. Use Tines’ Send Email action to configure email notifications.
2. Add an email inbox (e.g., from SquareX, Gmail) for testing purposes.
3. Test the email connector to confirm alerts can be sent successfully.

![image](https://github.com/user-attachments/assets/f40b0072-04b4-4880-bb1d-9c23e024ec8a)

![image](https://github.com/user-attachments/assets/62c39021-0c85-4480-8296-0465d027ab95)

**Step 7: Create User Prompt for Response**
1. In Tines, add a User Prompt action to allow the analyst to decide whether to isolate the machine.
2. Configure the prompt to display relevant alert information retrieved from the webhook.
3. Link the response options (Yes or No) to the appropriate next steps in the workflow.
![image](https://github.com/user-attachments/assets/240c0499-2571-4f4d-a6a2-96bc6eadc7d1)

**Step 8: Automate Machine Isolation**
1. Retrieve LimaCharlie API keys by following the LimaCharlie API Key Setup Guide.
2. Use Tines’ HTTP Request Action to send a POST request to LimaCharlie’s API to isolate the machine.
3. Parse the response to confirm the isolation status and update the workflow accordingly.
4. Add a final Slack Notification action to confirm the isolation.

