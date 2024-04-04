# Technologies for building a Website on AWS

Lets consider the various technologies to build a website on AWS.  
By website we mean, a frontend, a backend and a database.

For Frontend, we have various technologies and the choice basically falls on developer expertise. Comparing the current relevant for modern web technologies


## Frontend
Frontend is the presentation layer, basically the HTML, CSS and JavaScript  

##### React.js
* React.js, developed by Facebook, is a popular JavaScript library for building user interfaces.
* It follows a component-based architecture, making it easy to develop interactive and reusable UI components.
* **React.js is known for its virtual DOM, which provides efficient rendering and performance optimizations.**
* It has a vast ecosystem with libraries like React Router for routing and Redux for state management.

##### Vue.js
* Vue.js is a progressive JavaScript framework for building user interfaces, designed to be incrementally adoptable.
* It provides a flexible and approachable syntax, making it easy to integrate into existing projects.
* Vue.js offers a reactive data binding system and a component-based architecture similar to React.js.
* **It has a smaller learning curve compared to other frameworks**, making it suitable for smaller projects or developers transitioning from other frameworks.

##### Angular
* Angular is a comprehensive JavaScript framework maintained by Google for building large-scale, feature-rich web applications.
* It provides a complete solution with features like dependency injection, routing, forms, and state management out of the box.
* **Angular uses TypeScript**, a superset of JavaScript with static typing, which helps catch errors early in the development process.
* It follows a component-based architecture and utilizes a hierarchical dependency injection system.

##### Svelte
* Svelte is a relatively newer JavaScript framework that shifts the focus from runtime libraries to compile-time framework.
* **It compiles components into highly optimized JavaScript code during the build process, resulting in smaller bundle sizes and better performance.**
* Svelte's approach simplifies the development process by eliminating the need for virtual DOM diffing at runtime.
* It offers a reactive data binding system and a simple syntax for building components.

##### Bootstrap
* Bootstrap is a popular CSS framework for building responsive and mobile-first web projects.
* It provides a set of pre-designed UI components, such as buttons, forms, navigation bars, and grids, making it easy to create visually appealing interfaces.
* Bootstrap offers extensive documentation and customization options, allowing developers to create custom themes and layouts.

##### Tailwind CSS
* Tailwind CSS is a utility-first CSS framework that provides low-level utility classes for styling HTML elements.
* It offers a highly customizable and expressive approach to styling, allowing developers to create unique designs without writing custom CSS.
* Tailwind CSS encourages a functional and modular approach to styling, making it suitable for building scalable and maintainable UIs.

I didn't mention JSP (JavaServer Pages) or PHP in the list of frontend technologies. That's because JSP and PHP are primarily server-side technologies used for generating dynamic HTML content and processing server-side logic, rather than frontend technologies used for building interactive user interfaces in the browser.

While JSP and PHP are essential technologies for building dynamic web applications, they are often used in conjunction with frontend technologies (HTML, CSS, JavaScript) to create full-stack web applications. Frontend technologies handle the user interface and client-side interactions, while server-side technologies like JSP and PHP handle server-side logic, data processing, and generating dynamic content to be served to the client.

The choice, in the end, depends on developer preferences, and expertise in the team


##  Database
While Postgres is a no-brainer for database when you have a relational data model, the choice comes down to self-hosting it on EC2 instances or using **managed** AWS offerings like RDS or Aurora. The choice depends on developer maturity and scalability requirements.

For services like heartbeat monitoring, which do not require a normalized database, AWS DynamoDB is an excellent choice in terms of maintenance, reliability and scaling (be careful to avoid scans though)


### Backend Technologies

For backend technologies, we have the option to either
1. Use S3 website hosting (with Cloudfront CDN for serving HTTPS traffic), and using 
    * API Gateway + Lambda or VTL for backend or
    * AppSync for backend
2. Containerize your application code and host it via 
* ALB + ECS or
* AWS Elastic BeanStalk (with load balancing enabled)
* Amazon Lightsail.

Lets consider the pro-and-cons of S3 website hosting with API Gateway + Lambda for backend  

| Pros | Cons |
|---|---|
99.999999999% durability, 99.99% availability | S3 Static website hosting doesnot support HTTPS, you need to front that with Cloudfront CDN for HTTPS support | 
Truly serverless, literally no servers to manage | API Gateways have a timeout of 29 seconds |
Scalable by design | If you have a very high traffic website, lambda costs can go substantially up, workaround is to use VTL, but then for PUT requests you cannot do data validations |

Lets consider the pro and cons of containerizing your application.  
When containerizing the application we can use ElasticBeanStalk which can automatically handle the deployment details of capacity provisioning, load balancing, automatic scaling, and web application health monitoring.

| Pros | Cons |
|---|---|
99.9% availability | You need artifactory to store your war files or ECR to host your docker images | 
Supports applications developed in Java, .NET, PHP, Node.js, Python, Ruby and Go. | Deployments can be slow, it can take five minutes at least, and sometimes stretch to fifteen |
Theoretically Can support higher  RPS and timeouts than API g/w | You need to pre-configure public facing VPC and take care of security groups|

When choosing containerization we must choose between SpringBoot(using war files) and nodejs (in docker images)

**Spring Boot is the better option if you have compute-heavy webapp (Java based multi-threaded programming), while nodejs is better for real time capabilities in thin backend servers that perform little to no CPU intensive tasks (mostly I/O).**


Spring is a framework that tends to be closely to the server side, storage, threading, business logic, API development. So to develop a webapp in Spring Boot, you will use MVC where the View is built in some frontend like react


**Things I would never do in NodeJs:**
* Applications with a lot of business logic and validations that can be modeled with a strong type system or with OOP programming.
* Any sort of system that requires cpu intensive tasks like finding a the n-th prime or performing a Dijkstra run on a big matrix.
* Anything that requires multithreading and process orchestration.


### CI/CD
For deployment we have various options available under the CI/CD umbrella but lets consider first what is CI/CD

##### Continuous Integration (CI)
When modifications are made to a code repository (eg. Git), CI inspects each team member’s code to ensure smooth compatibility with the existing codebase. It detects new code changes automatically and initiates a build process that includes **code compilation, automated testing, and extensive checks and validations**.

##### Continuous Deployment/Delivery (CD)
After the code successfully passes all checks and tests in the CI phase, we can now go ahead with the deployment process.


##### Github Actions
Github actions is more focussed on CI, where in u build, test, run unit tests, lint code etc.  
After that for CD, u need to run either AWS commands to deploy infrastructure or there must be terraform or cloudformation templates to do the actual deployment on the cloud

This, in my view, is not infrastructure as code, but rather infrastructure as config

With Github Actions, to deploy your code on AWS, API calls to AWS need to be signed with credential information, so when you use one of the AWS SDKs or an AWS tool, you must provide it with AWS credentials and and AWS region. One way to do that in GitHub Actions is to use a repository secret with IAM credentials, but this doesn't follow AWS security guidelines on using long term credentials.  
Instead, we can use Using GitHub's OIDC provider (AssumeRoleWithWebIdentity) using this [github action]((https://github.com/aws-actions/configure-aws-credentials)


##### AWS Codepipeline
AWS CodePipeline is a fully managed continuous integration and continuous delivery (CI/CD) service provided by Amazon Web Services (AWS). It automates the build, test, and deployment phases of your release process, allowing you to quickly and reliably deliver updates to your applications and infrastructure.

We use Github hooks to trigger the CodePipeline, which is then triggers a pre-defined series of stages and actions to orchestrate your CI/CD workflow.  
Each stage represents a phase in your release process (e.g., source, build, test, deploy), and actions within stages perform specific tasks such as fetching source code, running tests, or deploying applications.

