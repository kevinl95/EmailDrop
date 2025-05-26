# EmailDrop: Postmark to Google Drive Attachment Uploader

This application automatically uploads email attachments from Postmark to Google Drive. It uses AWS Lambda, API Gateway, and Secrets Manager to handle the OAuth flow and file uploads.

## Architecture

- **API Gateway**: Provides endpoints for OAuth callback and Postmark webhook
- **Lambda Functions**: Handle OAuth flow and attachment uploads
- **Secrets Manager**: Securely stores OAuth refresh tokens
- **Google Drive API**: Destination for email attachments

## Setup Instructions

### 1. Google Drive API Setup

#### 1.1 Go to Google Cloud Console
- Visit https://console.cloud.google.com/

#### 1.2 Create a Project
- Click the project dropdown → New Project
- Name it (e.g., PostmarkUploader)
- Click Create

#### 1.3 Enable the Google Drive API
- Go to APIs & Services → Library
- Search for Google Drive API
- Click Enable

#### 1.4 Configure the OAuth Consent Screen
- Go to APIs & Services → OAuth consent screen
- Choose External
- Fill in:
  - App name
  - Support email
  - Developer contact email
- Add yourself as a test user
- Click Save and Continue

#### 1.5 Create OAuth 2.0 Credentials
- Go to APIs & Services → Credentials
- Click + Create Credentials → OAuth client ID
- Choose Web application
- Skip the redirect URI for now — you'll add it after deploying the stack
- Click Create
- Save the Client ID and Client Secret

### 2. Deploy the CloudFormation Stack

#### 2.1 Launch the Stack
- Upload the `cloudformation.yml` file to CloudFormation
- Fill in the parameters:
  - **GoogleClientId**: (from step 1.5)
  - **GoogleClientSecret**: (from step 1.5)
  - **GoogleDriveFolder**: (optional) Name of the folder to store attachments
  - **LambdaTimeout**: (optional) Default is 60 seconds. Increase for large attachments (up to 900s)

#### 2.2 Add the Redirect URI
- Once the stack is deployed, go to the Outputs tab in CloudFormation
- Copy the OAuthURL output
- Return to Google Cloud Console → Credentials
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
- In the stack outputs, find the PostmarkWebhookURL
- Configure your Postmark server to send inbound webhooks to this URL
- Any attachments in emails received by Postmark will now be uploaded to your Google Drive

## Customization Options

- **GoogleDriveFolder**: Change the folder where attachments are stored
- **LambdaTimeout**: Adjust timeout for handling large attachments

## Troubleshooting

If attachments aren't being uploaded:
1. Check CloudWatch Logs for the Lambda functions
2. Verify the OAuth flow was completed successfully
3. Ensure Postmark is correctly configured to send webhooks

## Security Considerations

- The application uses AWS Secrets Manager to securely store OAuth tokens
- Only minimal Google Drive permissions are requested (drive.file)
- Consider adding additional security measures for production use