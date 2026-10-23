# DECISIONS.md

## 1. MongoDB Connection

The backend started, but MongoDB showed an `Invalid scheme` error.

I checked `index.js` and found that it reads the database URL from environment variables. There was no proper `.env` configuration, so I created `server/.env` with these values:

* `DATABASE_URL=mongodb://localhost:27017`
* `DB_NAME=EMS`
* `PORT=8000`

After that, the backend connected to MongoDB.

## 2. Starting the Backend

I wasn't sure if Nodemon was installed globally, so I tried `npx nodemon index.js`.

The server started, but the database connection still failed. This helped me understand that starting the backend doesn't mean the database connection is working too.

## 3. Dashboard Page

The dashboard wasn't opening properly. I checked the route and found a filename capitalization mismatch in the import.

I corrected the import path to match the actual filename. This is important for Docker and Linux because they treat uppercase and lowercase letters differently.

## 4. Adding an Employee

When I tried to add an employee, the request failed with a 500 error, and nothing appeared in MongoDB.

I checked the browser's Network tab and then looked at the backend route in `server/routes/createEmployee.js`. The catch block was returning a generic error instead of showing the actual problem.

## 5. Form Was Not Sending Data

The problem was in `ModelDetails.jsx`.

The fetch request wasn't sending the form data to the backend. The inputs were also using `onSubmit` instead of `onChange`.

I fixed the input handlers and added the form data to the request body using `JSON.stringify(values)`. After fixing these issues, the backend could receive the employee details.

## 6. Dashboard Returned 404

Opening `/dashboard` directly caused a 404 from Nginx.

The problem wasn't necessarily React Router. Nginx was looking for a file or folder named `dashboard`, which didn't exist.

The Nginx configuration needed to return `index.html` for frontend routes so React Router could handle them.

## 7. Localhost and Ports

During local testing, the backend ran on port `8000` and the React frontend ran on port `5176`.

Both servers needed to be running.

One thing I learned is that `localhost` means the machine making the request. When the frontend runs in a browser, `localhost:8000` points to the user's computer, not automatically to a remote backend.

## 8. Docker and API Requests

The frontend had several hardcoded `localhost:8000` URLs. These wouldn't work correctly for a deployed application.

I changed the API requests to use `/api` and configured Nginx to forward them to the backend.

## 9. Jenkins and Docker Hub

Jenkins builds the frontend and backend images and pushes them to Docker Hub.

Both images currently use the `latest` tag. It works for this project, but it makes it harder to identify a particular build or return to an older image.

Unique image tags can be added later.

## 10. Jenkins on Windows

Jenkins currently runs on Windows, so the pipeline uses `bat` commands.

When I move Jenkins to Linux, these commands will need to change to `sh`.

I also tried using Trivy for image scanning, but the stage was commented out after scan failures. That still needs to be fixed.

## Things to Improve Later

* Move Jenkins to Linux.
* Use unique Docker image tags.
* Fix and enable the Trivy scan.
* Keep database passwords and other secrets out of the source code.
* Test the frontend routes and API requests in the remote deployment.
