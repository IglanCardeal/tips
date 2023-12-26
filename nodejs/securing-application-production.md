# Best Practices for Securing Node.js Applications in Production

## 1. Never Run Node.js With Root Privileges

If you instead run Node.js with root privileges, any vulnerabilities in your project or its dependencies can potentially be exploited to gain unauthorized access to your system. For example, an attacker could harness them to execute arbitrary code, access sensitive files, or even take control of the entire machine. Thus, using root users for Node.js must be avoided.

The best practice here is to create a dedicated user for running Node.js. This user should have only the permissions required to launch the app. This way, attackers who succeed in compromising your backend will be restricted to that user’s privileges, limiting the potential damage they can cause.

## 2. Keep Your NPM Libraries Up To Date

Use `npm audit` or `yarn audit` to check packages vulnerabilities.

## 3. Avoid Using Default Cookie Names

The cookie names used by your Node.js application can unintentionally reveal the technology stack your backend is based on. That is valuable information that you should always obscure, as attackers can use it again you. By knowing what framework you are using, they can exploit specific weaknesses associated with it.

## 4. Set the Security HTTP Headers

The default HTTP headers in Express are not very secure. We check header security with the online [Security Headers project](https://securityheaders.com/).

Some of these headers contain information that should not be publicly exposed, such as **X-Powered-By**. Others are missing and should be added to deal with various security-related aspects, including preventing cross-site scripting (XSS) attacks.

```js
const express = require('express')
const helmet = require('helmet')

const app = express()

// register the helmet middleware
// to set the security headers
app.use(helmet())
```

The `helmet()` middleware automatically removes unsafe headers and adds new ones, including **X-XSS-Protection**, **X-Content-Type-Options**, **Strict-Transport-Security**, and **X-Frame-Options**. These enforce best practices and help protect your application from common attacks.

The headers set by your Node.js app will now be considered secure.

## 5. Implement Rate Limiting

The easiest way to implement rate limiting is through the **rate-limiter-flexible** library. This dependency provides a configurable middleware to restrict the number of requests coming from the same IP address or user within a specified time frame.

```js
const express = require('express');
const { RateLimiterMemory } = require('rate-limiter-flexible');

const app = express();

const rateLimiter = new RateLimiterMemory({
  points: 10, // maximum number of requests allowed
  duration: 1, // time frame in seconds
});

const rateLimiterMiddleware = (req, res, next) => {
   rateLimiter.consume(req.ip)
      .then(() => {
          // request allowed,
          // proceed with handling the request
          next();
      })
      .catch(() => {
          // request limit exceeded,
          // respond with an appropriate error message
          res.status(429).send('Too Many Requests');
      });
   };

app.use(rateLimiterMiddleware);
```

## 6. Ensure Strong Authentication Policies

To protect your Node.js application against attacks that exploit user authentication, you need to enforce strong authentication policies. First, invite users to **set strong passwords**. Also, you should support **Multi-Factor Authentication** (MFA) and **Single Sign-On(SSO)**. MFA adds an extra layer of security by requiring users to provide multiple forms of authentication, while SSO simplifies the authentication process and reduces the risk of weak or reused passwords.

When it comes to hashing passwords for storage, prefer strong cryptographic functions like bcrypt over the methods offered by the Node.js crypto library. That package provides a secure password-hashing algorithm that makes it significantly harder for attackers to crack passwords. Finally, mitigate brute-force attacks by restricting the number of failed login attempts through rate limiting as explained earlier.

## 7. Do Not Send Unnecessary Info

Any information provided to the attacker unintentionally can be used against you. For this reason, server responses should contain only what the caller strictly needs. For example, avoid returning detailed error messages or stack traces directly to the client. Instead, provide generic error messages that do not reveal specific implementation details. The easiest at to do so is to run Node.js in production mode setting the NODE_ENV=production env, otherwise Express will add the stack trace in the error responses.

Similarly, you must be careful about the data included in API responses. Return only necessary data fields and avoid exposing sensitive information not requested by the caller. This will minimize the risk of accidentally disclosing confidential or privileged information.

## 8. Monitor Your Backend

Your backend in production may be under attack, and you may not even be aware of it. Here is why it is essential to monitor your Node.js application. By connecting it to an Application Performance Monitoring (APM) tool, you can keep track of it to identify security issues and ensure its overall health.

Fortunately, there are several APM libraries and services available for Node.js. Some of the most popular are SigNoz, Sentry, Prometheus, New Relic, and Elastic These provide information on various aspects of the application, including performance, error rates, resource usage, and security-related metrics. In particular, they enable real-time data collection and detection of anomalies or suspicious activity that could indicate a security breach. Some of them also offer osservability features to also track the deployment workflow in your CI/CD pipeline.

## 9. Adopt an HTTPS-Only Policy

By ensuring that your backend is accessible only via HTTPS, you will improve the confidentiality of data exchanged between clients and your Node.js server. HTTPS establishes an encrypted channel that safeguards sensitive information like passwords, session tokens, and user data from interception.

As part of such policy, you should also use HTTPS cookies. To achieve that, make sure that any cookies set by your Node.js application are marked as secure and httpOnly`:

```js
res.cookie('myCookie', 'cookieValue', {
  // create an HTTPS cookie
  secure: true,
  httpOnly: true,
});
```

Unintended parties or scripts will no longer be able to access your cookies. Plus, they will be transmitted exclusively over HTTPS connections.

## 10. Validate User Input

## 11. Use Security Linters

Security linters analyze your codebase to identify vulnerabilities, unsafe code sections, and best practice violations. One of the most popular is **eslint-plugin-security**, a set of ESLint rules to enforce security development in Node.js.

By integrating such tools into your development workflow, you can spot and address security issues early on. Specifically, they reduce the risk of introducing vulnerabilities into your application while coding. These tools particularly effective when added integrated in the CI/CD pipeline.

## 12. Prevent SQL Injection

SQL injection is a common security vulnerability that occurs when an attacker can manipulate input data passed into an SQL query. This generally happens when concatenating user input directly into SQL queries. In that scenario, attackers can forge specific inputs aimed at executing arbitrary SQL code, leading to unauthorized access and data breaches.

There are several ways to prevent SQL injection in Node.js. The most popular ones are:

- Use Prepared Statements or Parameterized Queries: These techniques involve separating the SQL code from the user input, preventing it from being interpreted as part of the query.
- Input Sanitization: Validate user input to reject malicious data, reducing the risk of SQL injection attacks.
- Use an ORM: ORM technologies like Sequelize generally provide built-in protection against SQL injection.

## 13. Limit Request Size

The default request body size limit in Node.js is 5 MB. To protect your backend from DDoS attacks where malicious users try to flood your server with data, it is recommended to reduce that limit. To do so, you can use the body-parser library and configure it as below:

```js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();

// set the request size limit to 1 MB
app.use(bodyParser.json({ limit: '1mb' }));
```

Adjust the value according to your specific requirements. Now, requests with a body larger than 1 MB will now be blocked immediately, preventing the server from allocating resources to process them.

## 14. Detect Vulnerabilities Through Automated Tools

Automated vulnerability scanning tools, such as **SonarQube** or similar, are valuable resources for identifying security problems in Node.js applications. These tools perform comprehensive scans of the codebase, dependencies, configurations, and other components to identify security weaknesses.

Here are some key benefits of using automated vulnerability scanners:

- Early Detection: Proactively identify security issues before deploying your application.
- Increased Coverage: Perform in-depth scans of all project files, ensuring high security coverage.
- Continuous Monitoring: Integrate them into your CI/CD pipeline to ensure that any new vulnerabilities introduced through code changes are promptly discovered.
