Authentication Project with Local and Google OAuth2 Strategies
This project demonstrates a Node.js and Express application with dual authentication methods: local username/password and Google OAuth2. Users can register and log in with either method, with sessions managed via express-session and authentication handled by Passport.js. This setup securely manages user sessions and leverages Google’s OAuth2 for social login.

Project Overview
Local Authentication: Users register with a username (email) and password. Passwords are hashed with bcrypt before storage in the PostgreSQL database.
Google OAuth2: Users can log in with their Google account. This method retrieves basic profile information and email, reducing the need for password management.
Session Management: User sessions are securely managed with express-session, allowing persistent login states across requests.
Environment Variables: Sensitive information (e.g., database credentials, OAuth secrets) is stored in an .env file and accessed through the dotenv package to enhance security.
Technologies Used
Node.js and Express: For server setup and routing.
PostgreSQL: For data storage, managed with the pg library.
Passport.js: For user authentication (local strategy and Google OAuth2).
express-session: To manage user sessions.
dotenv: For securely managing environment variables.
Setup Instructions
Clone the Repository:

bash

git clone <repository-url>
cd <repository-name>
Install Dependencies:


npm install
Environment Configuration:

Create a .env file in the root directory.
Add the following environment variables:


SESSION_SECRET=your-session-secret
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
PG_USER=your-database-username
PG_PASSWORD=your-database-password
PG_HOST=your-database-host
PG_PORT=your-database-port
PG_DATABASE=your-database-name
Run the Application:

sql

npm start
Access the Application:

Open your browser and go to http://localhost:3000.
Test the local registration, login, and Google login flows.
Code Explanation
Local Authentication (passport-local):

Users register and log in with their email and password.
Passwords are hashed with bcrypt and stored securely in PostgreSQL.
Google OAuth2 Authentication (passport-google-oauth2):

Users can log in with their Google accounts, which sends a profile and email to the application.
If a new user logs in with Google, their profile is stored in the database for future authentication.
Session Management:

Managed with express-session, which keeps users logged in across multiple requests.
Environment Variables:

All sensitive data is stored in the .env file, loaded at runtime using dotenv.
Example Code Snippets
Sample of Environment Variable Setup
plaintext
Copy code
SESSION_SECRET=your-session-secret
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
PG_USER=your-database-username
PG_PASSWORD=your-database-password
PG_HOST=your-database-host
PG_PORT=your-database-port
PG_DATABASE=your-database-name
Express Session Setup
javascript
Copy code
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: true,
  })
);
Google OAuth2 Strategy
javascript
Copy code
passport.use(
  "google",
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: "http://localhost:3000/auth/google/secrets",
      userProfileURL: "https://www.googleapis.com/oauth2/v3/userinfo",
    },
    async (accessToken, refreshToken, profile, cb) => {
      const result = await db.query("SELECT * FROM authen WHERE email = $1", [
        profile.email,
      ]);
      if (result.rows.length === 0) {
        const newUser = await db.query(
          "INSERT INTO authen (email, password) VALUES ($1, $2)",
          [profile.email, "google"]
        );
        return cb(null, newUser.rows[0]);
      } else {
        return cb(null, result.rows[0]);
      }
    }
  )
);
Routes
/register - Register with email and password
/login - Login with email and password
/auth/google - Initiates Google OAuth login
/auth/google/secrets - Callback route for Google OAuth
/logout - Logs the user out and redirects to home
Troubleshooting
OAuth Redirect URI Mismatch: Ensure that the callback URL in the Google Console matches the /auth/google/secrets route exactly.
Database Connection: Verify PostgreSQL credentials in .env and ensure the database is running.