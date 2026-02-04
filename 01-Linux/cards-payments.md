# Linux Security BaseLine: Cards and Payments (SQB Mobile)
## Business Context
Cards and Payments modules handle financial transactions in SQB Mobile (B2C) and SQB Business (B2B). 
Unathorized access may lead to financial lose.

## Security Goals
- Service isolation
- Protection of sensitive configuration
- Audit logging 
- Least previlage access

## Implementation
- Separate Linux users for cards and payments services
- Isolated directories under /opt/sqb
- Secure config files (chmod 600)
- Centrilized logs under /var/log/sqb
- Firewall with default deny policy

## Threat Mitigation
This baseline mitigates risks of lateral movement between services,
unauthorized access to sensitive configuration files,
and uncontrolled exposure of backend services.

## Validation
Access restrictions were verified by attempting cross-service access.
The Payments service user was unable to access Cards service directories
and configuration files, confirming proper isolation.

## Result
Reduced blast radius and improved auditability for banking services.
