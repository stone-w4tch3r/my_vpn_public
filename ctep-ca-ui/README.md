# About

The project is a web-based interface for managing a smallstep `step-ca` Certificate Authority.
In other words, it's shell wrapper for `step-ca` CLI.
It provides a user-friendly way to handle certificate operations, view logs, and manage server access.
The system is designed with security in mind, separating components with different privileges and logging all actions
for audit purposes. Written in python.

### Main Functions:

#### Certificate Management:

All on one page
- List certificates
- Generate new certificates (with options for key type, duration, etc.)
- Revoke certificates
- Renew certificates

#### Logging and Monitoring:

- View logs with filtering
- View command execution history
- Every action such as certificate generation, revocation etc shows it's related logs immediately to user in the UI
- Every action shows shell command that it's will execute

#### Web UI:

- Dashboard + Certificate Management Page
- Logs and Command History Page

### Security Features:

- Separation of privileged sudo setup process from main application
- Main process is run with special linux user with limited permissions
- Web UI is in separate unprivileged process
- Input value restrictions (eg letters and numbers only for key name)
- Input sanitization in CLIWrapper to prevent command injection
- Use of subprocess module with shell=True for secure command execution
- Logging of all command executions for audit purposes
- Configuration loaded from a separate file, not hardcoded

### Additional Notes:

- The system uses Flask as the web framework
- Type hints/dataclasses are used throughout for better code clarity and error checking
- The design allows for easy extension and maintenance
- The AutoSetup component (for initial installation and setup) is designed to be run separately with elevated privileges
- Web app auth is provided via third-party proxy, not required for this project

---
# Architecture

### Overview
```puml
@startuml
!define RECTANGLE class
!define DOCKER_NETWORK frame

skinparam componentStyle uml2

actor "User" as user
cloud "Internet" as internet

rectangle "Host System" {
  [Bunkerweb] as bunkerweb
  [Caddy] as caddy
  [Authelia] as authelia
  
  DOCKER_NETWORK "Docker Network" {
    package "Web Frontend Container" as webfront {
      [Flask Web Server]
    }
    
    package "MainApp Container" as mainapp {
      [Main Application]
      [API Server]
      [CLI Wrapper]
    }
  }
  
  [step-ca CLI] as stepcacli
}

user -down-> internet
internet -down-> bunkerweb
bunkerweb -down-> caddy
caddy -down-> authelia
authelia -down-> webfront

webfront -down-> mainapp : mTLS & authenticated API calls

mainapp -down-> stepcacli : Limited sudo access

note right of caddy : HTTPS & Reverse Proxy
note right of authelia : Authentication
note right of webfront : Runs without permissions
note right of mainapp : Runs with limited permissions & AppArmor/SELinux profile
note right of stepcacli : Installed globally on host
@enduml
```

### Old design
```puml
@startuml
skinparam classAttributeIconSize 0

class FlaskWebServer {
  +run()
  +handle_request()
}

class MainApplication {
  -config: Config
  -cli_wrapper: CLIWrapper
  -logger: Logger
  +list_certificates()
  +generate_certificate()
  +revoke_certificate()
  +renew_certificate()
  +view_logs()
  +view_command_history()
}

class CLIWrapper {
  -sanitize_input(input: str): str
  +execute_command(command: str): str
}

class Config {
  +load_config()
  +get_config_value(key: str): Any
}

class Logger {
  +log_action(action: str, details: dict)
  +get_logs(filter: dict): List[LogEntry]
}

class CertificateManager {
  +list_certificates(): List[Certificate]
  +generate_certificate(options: CertOptions): Certificate
  +revoke_certificate(cert_id: str)
  +renew_certificate(cert_id: str): Certificate
}

class AutoSetup {
  +run_setup()
  -setup_sudo_permissions()
  -create_limited_user()
}

FlaskWebServer --> MainApplication : uses
MainApplication --> CLIWrapper : uses
MainApplication --> Config : uses
MainApplication --> Logger : uses
MainApplication --> CertificateManager : uses
CertificateManager --> CLIWrapper : uses
AutoSetup --> CLIWrapper : uses

@enduml
```

### Initial design
```puml
@startuml

folder "step-ca-shared" {
  folder "nested" {
    class Foo {
    }
  }
  class CLIWrapper {
    -execute_command(command: str): Dict
    +run_step_ca_command(args: List[str]): Dict
    -sanitize_input(input: str): str
  }
}

package "step-ca-web" {
  class Main {
    +init_app()
    +index()
    +certificates()
    +server_config()
    +logs()
    +command_history()
  }

  class Config {
    +load_config()
    +get_setting(key: str): Any
    +update_setting(key: str, value: Any)
  }

  class Auth {
    +login_required(func)
    +verify_credentials(username: str, password: str): bool
  }

  class CertManager {
    +list_certificates(): List[Dict]
    +generate_certificate(params: Dict): Dict
    +revoke_certificate(cert_id: str): Dict
    +renew_certificate(cert_id: str): Dict
  }

  class ServerConfig {
    +get_config(): Dict
    +update_config(new_config: Dict): Dict
  }

  class LogHandler {
    +log_command(command: str, output: str, status: str)
    +get_logs(filter_params: Dict): List[Dict]
    +get_command_history(): List[Dict]
  }
}

package "step-ca-setup" {
  class AutoSetup {
    +run_setup_wizard(): Dict
    +generate_default_config(): Dict
    +install_step_ca(): Dict
  }
}

interface "Flask" as FlaskInterface

Main -up-> FlaskInterface : uses
Main --> Config : uses
Main --> Auth : uses
Main --> CertManager : uses
Main --> ServerConfig : uses
Main --> LogHandler : uses

CertManager --> CLIWrapper : uses
ServerConfig --> CLIWrapper : uses
CLIWrapper --> LogHandler : uses
AutoSetup --> CLIWrapper : uses

note right of CLIWrapper
  Shared component
  Executes step-ca CLI commands
  Returns log data for each command
end note

note right of LogHandler
  Stores logs in JSON format
  Provides access to command history
end note

note right of AutoSetup
  Separate component
  Requires sudo privileges
  Handles step-ca installation
end note

@enduml
```

---
# TODO
- TLS between containers
- API request ip whitelisting
- AppArmor/SELinux profiles
- Firewall between containers?
- API keys
- Keys secrets management
- Security events email alerts
- Read-only filesystem for containers