# MVP Scope

## Objective

The MVP will replace the current spreadsheet-based job application tracking process with an AI-assisted web application.

The product should allow a user to paste a job advertisement URL, automatically extract key job information, review the extracted result, save the opportunity, maintain its application status, and review basic application activity.

## In Scope

### Authentication

* Sign in using a Google account
* Automatically create the OfferBuddy user profile after the first successful sign-in
* Maintain an authenticated session
* Sign out
* Access only the user's own application records


### AI-Assisted Job Capture

- Paste a job advertisement URL
- Validate the submitted URL
- Retrieve publicly accessible job advertisement content where permitted
- Send relevant job content to an AI provider for structured extraction
- Extract the job title
- Extract the company name
- Extract the location
- Extract the employment type when available
- Extract salary information when available
- Extract the job description
- Extract relevant skills or keywords where practical
- Identify the source platform where practical
- Display extracted information as an editable draft
- Allow the user to correct or replace extracted information
- Save the original job advertisement URL
- Support manual data entry when retrieval or AI extraction fails

### Job Parsing Status

The system should distinguish between parsing outcomes such as:

- Pending
- Processing
- Completed
- Partially Completed
- Failed
- Manual Entry

A parsing failure must not prevent the user from saving the application manually.

### Application Tracking

- Create a job application record
- Update the application status
- Record the application date
- Add notes
- View application details
- Edit an existing application
- Archive or delete an application

### Application Statuses

Initial statuses:

- Saved
- Applied
- Screening
- Interview
- Offer
- Rejected
- Withdrawn

### Application List

- View all applications
- Filter by status
- Search by company or job title
- Sort by application date or update date

### Basic Dashboard

- Total applications
- Applications by status
- Applications created during a selected period
- Interview count
- Offer count

## Out of Scope

The following capabilities are not part of the first MVP:

- Automatic job applications
- Large-scale job board crawling
- Browser extension
- Automatic email status detection
- Resume generation
- Resume optimisation
- Resume-to-job matching
- AI mock interviews
- Interview question generation
- Cover letter generation
- Voice interview calls
- Advanced job matching
- Multi-country localisation
- Native iOS or Android applications
- Team or recruiter accounts
- Subscription billing
- Microservices
- Kubernetes deployment
- Advanced analytics
- Social login

## MVP Success Criteria

The MVP will be considered successful when a user can:

1. Sign in using Google
2. Paste a supported job advertisement URL
3. Receive an editable draft containing extracted job information
4. Correct incomplete or inaccurate extracted values
5. Save the job as an application record
6. Manually enter information when extraction fails
7. Update the application through its lifecycle
8. Search and filter existing applications
9. View basic application statistics
10. Use OfferBuddy instead of the existing spreadsheet for normal job tracking

## Constraints

- Development time is limited
- The product is initially being developed by one engineer
- The first release should minimise infrastructure cost
- The application should be deployable using Docker
- The architecture should support future growth without introducing unnecessary complexity
- AI output may be incomplete or inaccurate.
- External job websites may block or limit automated retrieval.
- Job website HTML structures may change without notice.
- The core workflow must remain usable when the AI provider is unavailable.
- AI usage should be designed with cost limits and response-size controls.
