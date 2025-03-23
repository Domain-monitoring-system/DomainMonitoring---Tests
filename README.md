# Domain Monitoring System - Test Suite

This repository contains the test framework for the Domain Monitoring System, including Selenium-based functional tests and Locust performance tests.

## Overview

The test suite verifies functionality across several key areas:
- User authentication (registration, login, logout)
- Domain management (adding, deleting, refreshing domains)
- Scheduler functionality (hourly/daily checks)
- Load testing using Locust

## Repository Structure

```
domain-monitoring-tests/
├── conftest.py                # Test configuration and logging setup
├── test_base.py               # Base test class with common functionality
├── test_auth.py               # Authentication tests
├── test_domains.py            # Domain management tests
├── test_scheduler.py          # Scheduler tests
├── locustfile.py              # Performance/load testing with Locust
├── run_tests.py               # Test runner script
├── requirements.txt           # Python dependencies
└── README.md                  # This readme file
```

## Test Categories

### Authentication Tests (`test_auth.py`)

Tests user registration, login, and logout functionality:
- `test_registration_success`: Verifies new user registration
- `test_login_success`: Tests login with valid credentials
- `test_logout`: Confirms logout functionality

### Domain Management Tests (`test_domains.py`)

Tests domain monitoring functionality:
- `test_add_domain`: Adds single domains
- `test_delete_domain`: Removes domains
- `test_refresh_domains`: Tests domain status refresh
- `test_file_upload`: Tests bulk domain addition via file upload

### Scheduler Tests (`test_scheduler.py`)

Tests scheduling functionality:
- `test_hourly_schedule`: Sets up hourly domain checks
- `test_daily_schedule`: Sets up daily domain checks
- `test_stop_schedule`: Tests stopping scheduled checks

### Performance Tests (`locustfile.py`)

Load testing using Locust:
- Tests API performance under load
- Monitors for performance degradation
- Simulates multiple concurrent users

## Setup

### Prerequisites

- Python 3.8 or higher
- Chrome browser and ChromeDriver
- Access to the Domain Monitoring System application

### Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd domain-monitoring-tests
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Configure the environment:
   ```bash
   # Set the application URL (default: http://host.docker.internal:8080)
   export APP_URL=http://localhost:8080
   ```

## Running Tests

### Running All Tests

Execute the test runner script:
```bash
python run_tests.py
```

This will run the core test suite defined in `run_tests.py`.

### Running Specific Test Categories

To run only authentication tests:
```bash
python -m unittest test_auth.py
```

To run only domain management tests:
```bash
python -m unittest test_domains.py
```

To run only scheduler tests:
```bash
python -m unittest test_scheduler.py
```

### Running Individual Tests

To run a specific test method:
```bash
python -m unittest test_auth.AuthenticationTests.test_login_success
```

### Running Performance Tests

To run the Locust performance tests:
```bash
locust -f locustfile.py --host=http://localhost:8080
```

Then open http://localhost:8089 in your browser to access the Locust web interface.

## Test Framework Architecture

### Test Base

The `BaseTest` class in `test_base.py` provides common functionality:
- Browser setup and teardown
- Helper methods for finding elements
- Alert handling
- Test user creation

### Logging

The test suite uses a configured logger (`conftest.py`) that:
- Logs to both console and file
- Creates daily log files in the `test_logs` directory
- Records different log levels for debugging

## Docker Integration

The tests are designed to run in both local and Docker environments:
- Chrome is configured to run in headless mode
- Tests can target a Docker container via `host.docker.internal`
- Configurable timeouts accommodate container networking

## Adding New Tests

### Creating a New Test Case

1. Create a new test file or add to an existing one
2. Inherit from `BaseTest` for common functionality
3. Use the logger for debugging and tracking
4. Follow the existing pattern for element interaction:
   ```python
   def test_new_functionality(self):
       logging.info("Starting new test")
       element = self.wait_for_element(By.ID, "element-id")
       element.click()
       # ... more test steps ...
       self.assertIn("Expected Result", result.text)
   ```

### Test Structure

Each test should:
1. Log the start of the test
2. Perform setup operations if needed
3. Execute the test steps
4. Verify the results with assertions
5. Log the test outcome

## Troubleshooting

### Common Issues

1. **Element not found exceptions**:
   - Increase the timeout in `wait_for_element()`
   - Check if the element ID or selector has changed
   - Verify the page is fully loaded before searching

2. **Test timing issues**:
   - Add explicit waits for dynamic elements
   - Use `WebDriverWait` for elements that take time to appear
   - Add small `time.sleep()` delays when necessary

3. **Browser or WebDriver issues**:
   - Update ChromeDriver to match your Chrome version
   - Try running with `--headless=new` instead of just `--headless`
   - Check Chrome's sandbox settings

### Debugging Tips

1. Use the logging system:
   ```python
   logging.debug("Detailed debug information")
   logging.info("General test progress")
   logging.error("Test failures or errors")
   ```

2. Review the log files in `test_logs/test_YYYYMMDD.log`

3. Add screenshots for failures:
   ```python
   def take_screenshot(self, name):
       self.driver.save_screenshot(f"screenshots/{name}.png")
   ```

## CI/CD Integration

The test suite is designed to integrate with CI/CD pipelines:

1. The tests run automatically during the CI/CD pipeline
2. Exit code 0 indicates success, non-zero indicates failure
3. Log files are generated for troubleshooting

Example Jenkins pipeline integration:
```groovy
stage('Functional Tests') {
    agent {
        docker {
            image 'python:3.9-slim'
            args '--network host'
        }
    }
    steps {
        sh '''
            pip install -r requirements.txt
            python run_tests.py
        '''
    }
    post {
        always {
            archiveArtifacts artifacts: 'test_logs/*.log', allowEmptyArchive: true
        }
    }
}
```