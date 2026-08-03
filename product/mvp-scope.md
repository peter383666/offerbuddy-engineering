# MVP Scope

## Objective

The MVP will replace the current spreadsheet-based job application tracking process with a simple web application.

The product should allow a user to save a job opportunity, maintain its application status, and review basic application activity.

## In Scope

### User Account

- Register an account
- Log in and log out
- Access only the user's own application records

### Job Capture

- Paste a job advertisement URL
- Enter or review the job title
- Enter or review the company name
- Record the job location
- Record the source platform
- Save the original job advertisement URL
- Store the job description where available
- Allow manual correction of extracted information

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
- AI mock interviews
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

1. Create an account
2. Paste or manually enter a job opportunity
3. Save the job as an application record
4. Update the application through its lifecycle
5. Search and filter existing applications
6. View basic application statistics
7. Use the application instead of the existing spreadsheet for normal job tracking

## Constraints

- Development time is limited
- The product is initially being developed by one engineer
- The first release should minimise infrastructure cost
- The application should be deployable using Docker
- The architecture should support future growth without introducing unnecessary complexity