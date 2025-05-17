# TechDesk Project

Welcome to the TechDesk project! This repository contains a fully functional ticketing system built to streamline support operations. The project consists of two main components: a robust backend service running inside a Docker container and a Java Swing desktop client provided as a JAR file. In addition, comprehensive API documentation, code documentation, and testing have been completed.

---

## Table of Contents

- [Overview](#overview)
  - [Backend Service](#backend-service)
  - [Database Design](#database-design)
  - [Java Swing Client](#java-swing-client)
  - [Documentation and Testing](#documentation-and-testing)
- [How to Run the Project](#how-to-run-the-project)
  - [Prerequisites](#prerequisites)
  - [Deploying with Docker Compose](#deploying-with-docker-compose)
    - [Dockerfile (Backend)](#dockerfile-backend)
    - [docker-compose.yml](#docker-composeyml)
    - [How to Start the Services](#how-to-start-the-services)
  - [Running the Java Swing Client](#running-the-java-swing-client)
  - [Viewing the Code Documentation](#viewing-the-code-documentation)
- [Additional Information](#additional-information)
  - [Using the SQL Script](#using-the-sql-script)
  - [Default Users and Credentials](#default-users-and-credentials)
- [Credits](#credits)

---

## Overview

The TechDesk project was developed to efficiently manage support tickets and related operations. Below are the key aspects of the work:

### Backend Service

- **Functionality:**  
  Handles RESTful API calls for managing users, tickets, comments, and audit logs.
- **Database Integration:**  
  Interacts with an Oracle database that stores critical tables such as `APP_USERS`, `TICKETS`, `COMMENTS`, and `TICKET_AUDIT_LOGS`.
- **Technology:**  
  Packaged as a Docker container for ease of deployment.

### Database Design

- **Schema Details:**  
  - Uses `RAW(16)` fields to store UUIDs, ensuring data integrity and uniqueness.
  - Enforces relationships through primary keys, foreign keys, and unique constraints.
- **Security:**  
  Implements role-based access control to differentiate access levels among administrators, support staff, and employees.

### Java Swing Client

- **User Interface:**  
  Provides an intuitive graphical interface for interacting with the backend services.
- **Delivery:**  
  Delivered as a JAR file for easy execution on any system with Java installed.

### Documentation and Testing

- **API Documentation:**  
  Full Swagger API documentation is available to detail the available endpoints.  
  Access it here: [Swagger API Documentation](http://localhost:8080/swagger-ui/index.html#/)
- **Code Documentation:**  
  Comprehensive Javadoc has been generated.  
  View it at: [Code Documentation](http://localhost:63342/TechDesk/target/reports/apidocs/com/techdesk/services/Impl/package-summary.html)  
  *Note:* The port (`63342`) is provided by IntelliJ.

---

## How to Run the Project

Follow these steps to set up and run the TechDesk project in your environment.

### Prerequisites

- **Docker:**  
  Ensure Docker is installed on your system. Download it from [Docker's official site](https://www.docker.com/get-started).

- **Java:**  
  Make sure Java JDK 8 or higher is installed. Download it from [Oracle's Java downloads](https://www.oracle.com/java/technologies/javase-downloads.html).

---

### Deploying with Docker Compose

This project provides a turnkey Docker setup to deploy both the backend service and the Oracle XE database together. The following instructions guide you through building and running the Docker containers.

#### Dockerfile (Backend)

Create a file named `Dockerfile` in the root of your project (if not already present):

```dockerfile
# Use an OpenJDK 17 slim image (adjust if necessary)
FROM openjdk:17-jdk-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the Spring Boot JAR into the container (ensure your JAR is built and located in the target folder)
COPY target/techdesk-backend.jar app.jar

# Expose the application port (matches server.port in application.properties)
EXPOSE 8080

# Run the Spring Boot application
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

> **Note:** Build your backend JAR (e.g., using Maven or Gradle) so that it is available at `target/techdesk-backend.jar`.

#### docker-compose.yml

Create a file named `docker-compose.yml` in the project root with the following content:

```yaml
version: '3.8'

services:
  oracle-xe:
    image: container-registry.oracle.com/database/express:21.3.0-xe
    container_name: oracle-xe
    environment:
      - ORACLE_PWD=oracle
      - ORACLE_CHARACTERSET=AL32UTF8
    ports:
      - "1521:1521"
      - "5500:5500"
    healthcheck:
      test: ["CMD-SHELL", "echo 'SELECT 1 FROM DUAL;' | sqlplus system/oracle@localhost:1521/XEPDB1 || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 10
    volumes:
      # Mount the SQL script for database initialization from src/main/resources/db/script.sql
      - ./src/main/resources/db/script.sql:/app/script.sql

  techdesk-backend:
    build: .
    container_name: techdesk-backend
    ports:
      - "8080:8080"
    depends_on:
      oracle-xe:
        condition: service_healthy
    environment:
      # These values override the properties in your application.yml
      - SPRING_DATASOURCE_URL=jdbc:oracle:thin:@oracle-xe:1521/XEPDB1
      - SPRING_DATASOURCE_USERNAME=techdesk
      - SPRING_DATASOURCE_PASSWORD=techdesk_password
```

#### How to Start the Services

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/Elmehdi-Erraji/TechDesk.git
   cd TechDesk
   ```

2. **Build and Start the Containers:**

   Run the following command to build the backend image and start both the Oracle XE database and the backend service:

   ```bash
   docker-compose up --build
   ```

3. **Verify the Containers are Running:**

   Use the command below to check that both containers are up and running:

   ```bash
   docker ps
   ```

4. **(Optional) Initialize the Database:**

   If your backend does not automatically initialize the database schema (via `ddl-auto=update` or a migration tool), you can manually run the SQL script:

    - Enter the Oracle container:
      ```bash
      docker exec -it oracle-xe bash
      ```
    - Launch SQL*Plus:
      ```bash
      sqlplus system/oracle@XEPDB1
      ```
    - Run the script:
      ```sql
      @/app/script.sql
      ```
    - Exit SQL*Plus and the container:
      ```sql
      EXIT
      ```
      ```bash
      exit
      ```

5. **Access the Backend:**

   The backend service should now be accessible at [http://localhost:8080](http://localhost:8080).

---

### Running the Java Swing Client

1. **Obtain the Swing Client JAR:**

   Ensure that the provided Swing client JAR (e.g., `techdesk-swing.jar`) is in your working directory.

2. **Launch the Swing Client:**

   Open a terminal and run:

   ```bash
   java -jar techdesk-swing.jar
   ```

   This launches the Swing client, which will connect to the backend service at [http://localhost:8080](http://localhost:8080). Ensure that the Docker containers are running before starting the client.

---

### Viewing the Code Documentation

If you need to review the generated code documentation:

1. **Open Your Browser:**

   Navigate to the Javadoc index. For example, if you're using IntelliJ, you can access it at:

   [http://localhost:63342/TechDesk/target/reports/apidocs/com/techdesk/services/Impl/package-summary.html](http://localhost:63342/TechDesk/target/reports/apidocs/com/techdesk/services/Impl/package-summary.html)

   *Note:* The port (`63342`) is provided by IntelliJ.

---

## Additional Information

### Using the SQL Script

The file `script.sql` contains the necessary statements to:

- Create the required tables (`APP_USERS`, `TICKETS`, `COMMENTS`, `TICKET_AUDIT_LOGS`, etc.).
- Insert sample data for testing, including default user accounts.

To execute this script manually:

1. Make sure you have mounted the script in the `docker-compose.yml` (if desired) or have it accessible in your container.
2. Connect to the Oracle XE container and run the script as shown in the [How to Start the Services](#how-to-start-the-services) section.

### Default Users and Credentials

Below are the default user accounts created by `script.sql` (if included in the script):

| Role     | Username   | Password      |
|----------|----------- |---------------|
| Admin    | `admin`    | `Password123` |
| Support  | `support`  | `Password123` |
| Employee | `employee` | `Password123` |

- **Admin** can view and  track changes in the audit log.
- **Support** can view and update tickets (comment, change status).
- **Employee** can create and edit their own tickets (subject to status restrictions) but cannot view system-wide data.

Feel free to modify these credentials in the `script.sql` to suit your security requirements.


## Want to Access the Code?

To maintain project security and code integrity, the source code is currently private.

🛠️ If you're interested in accessing the full source or discussing the project in more detail, feel free to reach out directly:

📩 **[elmehdi-erraji@hotmail.com](mailto:elmehdi-erraji@hotmail.com)**  
or  
💬 Connect with me on [LinkedIn](https://www.linkedin.com/in/mehdi-erraji)

---
---

## Credits

Thank you for taking the time to review and use the TechDesk project. Your feedback is highly appreciated. If you encounter any issues or have suggestions for improvements, please feel free to open an issue or contact me directly.

Enjoy working with TechDesk!

---
