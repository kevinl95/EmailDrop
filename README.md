# EmailDrop: Postmark to Google Drive Attachment Uploader

[![Lint CloudFormation Template](https://github.com/kevinl95/EmailDrop/actions/workflows/main.yml/badge.svg)](https://github.com/kevinl95/EmailDrop/actions/workflows/main.yml)

This application automatically uploads email attachments from Postmark to Google Drive. When emails are sent to your Postmark inbound email address, any attachments are automatically saved to your specified Google Drive folder. The system uses AWS Lambda, API Gateway, and Secrets Manager to handle the OAuth flow and file uploads.

## Architecture

- **Postmark**: Receives emails and sends attachment data via webhooks
- **API Gateway**: Provides endpoints for OAuth callback and Postmark webhook
- **Lambda Functions**: Handle OAuth flow and attachment uploads
- **Secrets Manager**: Securely stores OAuth refresh tokens
- **Google Drive API**: Destination for email attachments

## Setup Instructions

### 1. Google Drive API Setup

#### 1.1 Go to Google Cloud Console
- Visit [https://console.cloud.google.com/](https://console.cloud.google.com/)

#### 1.2 Create a Project
- Click the project dropdown → New Project
- Name it (e.g., PostmarkUploader)
- Click Create

#### 1.3 Enable the Google Drive API
- Go to **APIs & Services** → **Library**
- Search for Google Drive API
- Click Enable

#### 1.4 Configure the OAuth Consent Screen
- Go to **APIs & Services** → **OAuth consent screen**
- Choose External
- Fill in:
  - App name
  - Support email
  - Developer contact email (these can all be your personal email)
- Add yourself as a test user using the **Audience** page on the sidebar
- Click Save and Continue

#### 1.5 Create OAuth 2.0 Credentials
- Go to **APIs & Services** → **Credentials**
- Click **Create Credentials** → **OAuth client ID**
- Choose Web application
- Skip the redirect URI for now — you'll add it after deploying the stack
- Click Create
- Save the Client ID and Client Secret

### 2. Deploy the CloudFormation Stack

#### 2.1 Launch the Stack

[![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?templateURL=https://emaildroppostmark.s3.us-west-2.amazonaws.com/cloudformation.yml)

- Click the Launch Stack button
- Fill in the parameters:
  - **GoogleClientId**: (from step 1.5)
  - **GoogleClientSecret**: (from step 1.5)
  - **GoogleDriveFolder**: (optional) Name of the folder to store attachments
  - **LambdaTimeout**: (optional) Default is 60 seconds. Increase for large attachments (up to 900s)

#### 2.2 Add the Redirect URI
- Once the stack is deployed, go to the Outputs tab in CloudFormation
- Copy the OAuthURL output
- Return to Google Cloud Console → Client
- Click on your OAuth 2.0 client
- Edit the Authorized redirect URIs
- Paste in the URL you copied from the stack output
- Save changes

### 3. Complete the OAuth Flow
- In the stack outputs, find the OAuthURL
- Open it in your browser
- Approve access to Google Drive
- The Lambda will exchange the code for tokens and store them

### 4. Configure Postmark

#### 4.1 Create a Postmark Account
- Sign up at [postmarkapp.com](https://postmarkapp.com)
- Create a new server (or use an existing one)

#### 4.2 Set Up Inbound Email Processing
- In your Postmark dashboard, navigate to **Servers** → **Your Server Name** → **Default Inbound Stream**
- Navigate to **Setup Instructions**
- Note your unique inbound email address

#### 4.3 Configure Webhook URL
- From the **Setup Instructions** page, find the **Webhook URL** field
- In the CloudFormation stack outputs, find the **PostmarkWebhookURL** value
- Copy this URL and paste it into the Postmark Webhook URL field
- Save your changes

#### 4.4 Test the Integration
- Send an email with attachments to your Postmark inbound email address
- The attachments should be automatically uploaded to your Google Drive folder
- Check the Lambda logs in CloudWatch if you encounter any issues

## Customization Options

- **GoogleDriveFolder**: Change the folder where attachments are stored
- **LambdaTimeout**: Adjust timeout for handling large attachments

## Troubleshooting

If attachments aren't being uploaded:
1. Check CloudWatch Logs for the Lambda functions
2. Verify the OAuth flow was completed successfully
3. Ensure Postmark is correctly configured to send webhooks
4. Test your Postmark inbound email by sending a test email with attachments