# Email Sender Service (mock)

## Idea
Create an email service API that accepts email requests and logs them to files instead of actually sending emails. This teaches service-oriented architecture patterns and queue concepts without external dependencies.

## Learning Objectives
- Design a mail service API
- Implement email queue/job storage
- Handle async operations
- Work with email templates
- Build status tracking for sent emails

## Implementation Tips
- Accept email requests with to, subject, body fields
- Validate email addresses format
- Store emails in JSON queue files
- Add email templates support
- Implement email status tracking (pending, sent, failed)
- Add scheduling for email sending
- Log email sending to files
- Implement email history/audit
- Add retry logic for failed attempts
- Create email configuration options
- Add HTML email support
- Implement batch email sending

## Key Challenges
- Email validation and sanitization
- Template rendering
- Queue persistence and reliability
- Status tracking accuracy
- Error handling and retries
- Performance with large email volumes
