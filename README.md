# Smart Grid Manager API - Developer User Guide

## Introduction

Welcome to the Smart Grid Manager API! This guide will walk you through everything you need to know to interact with the API effectively. The Smart Grid Manager system provides a secure, privacy-focused solution that gives you complete control over your energy assets while ensuring reliable operation.

### What is Smart Grid Manager?

Smart Grid Manager is a sophisticated system designed to control and manage power in- and output in real-time across a wide variety of grid assets. It enables grid operators, asset owners, traders and energy companies to:

1. **Actively control energy production and consumption** from diverse grid assets
2. **Respond to grid demands** by adjusting power flow
3. **Schedule production and storage levels** throughout the day
4. **Ensure compliance** with grid regulations and power agreements

The system supports a wide range of grid-connected assets including:
- Solar installations (inverters, panels)
- Wind turbines
- Battery storage systems
- EV charging stations
- Conventional generators
- Data loggers and monitoring equipment

#### Key Capabilities

- **Real-time Control**: Adjust power flow of diverse grid assets instantly
- **Multi-Asset Support**: Manage solar inverters, wind turbines, battery systems, EV chargers, and more
- **Closed Loop Control**: Maintain precise setpoints automatically through PID feedback control
- **Scheduled Management**: Define operation parameters for specific time (UTC) periods across all asset types
- **Category-based Control**: Group devices by type for easier management (inverters, batteries, EV chargers, etc.)
- **Automated Failsafes**: Watchdog mechanism ensures devices revert to safe states
- **Comprehensive Logging**: Track all control activities and device errors across your asset portfolio
- **Flexible Authentication**: Control access at different levels with tokens
- **Protocol Flexibility**: Support for multiple communication protocols (Modbus, MQTT, proprietary APIs)
- **Edge Intelligence**: Local processing at the edge minimizes latency and ensures reliability

### Helin Platform Integration

The Helin Platform serves as the critical backbone between the Smart Grid Manager API and the edge nodes deployed at your sites:

- **Secure Communication**: All commands and data are transmitted securely between the API and edge nodes
- **Real-time Command Delivery**: The platform ensures curtailment commands reach edge nodes with minimal latency
- **Resilient Connection**: Edge nodes maintain local capabilities even during temporary connection issues
- **Data Synchronization**: Schedule information is automatically synchronized to edge nodes
- **Health Monitoring**: The platform continuously monitors the connection status of all edge nodes

Find your full Helin Platform reference at XX/YY 



## Table of Contents

0. [Introduction]()
1. [Understanding Smart Grid Manager API](#understanding-smart-grid-manager)
    - [System Architecture](#system-architecture)
    - [Command Processing System](#command-processing-system)
      - [API Setpoint Delivery Types](#setpoint-delivery-methods)
      - [Edge Setpoint Transfer](#setpoint-lifetime-management)
    - [API Setpoints Types in detail]()
       - [Immediate setpoint (% or kW)]()
       - [Scheduled setpoint (% or kW)]()
       - [Dynamic setpoint (kW)]()
       - [Fallback setpoint (% or kW)]()
    - [Edge Setpoint transfer in detail]()
       - [Cyclic write (default)]()
       - [Cyclic write for dynamic setpoint)]()
       - [Fallback / watchdog]()
    - [Key Capabilities](#key-capabilities)
    - [Common Use Cases](#common-use-cases)
2. [Authentication](#authentication)
   - [Obtaining an Auth0 Bearer Token](#obtaining-an-auth0-bearer-token)
   - [X-Management-Token](#x-management-token)
   - [Managing Site Management Tokens](#managing-site-management-tokens)
   - [Using the Authentication in Requests](#using-the-authentication-in-requests)
3. [Health and Version](#health-and-version)
   - [Check API Health](#check-api-health)
   - [Get Site Health Report](#get-site-health-report)
   - [Get API Version](#get-api-version)
4. [Site Management](#site-management)
5. [Node Management](#node-management)
6. [Device Management](#device-management)
7. [Control Operations](#control-operations)
8. [Logs](#logs)
9. [Schedules](#schedules)
10. [Best Practices](#best-practices)
    - [Supported Device Types](#supported-device-types)
      - [Solar Inverters](#solar-inverters)
      - [Battery Systems](#battery-systems)
      - [Communication & Data Loggers](#communication--data-loggers)
      - [Protocol Support](#protocol-support)
      - [Special Purpose Devices](#special-purpose-devices)
    - [Communication Protocols](#communication-protocols)
    - [Control Values](#control-values)
    - [Watchdog Behavior](#watchdog-behavior)
    - [Schedules](#schedules)
    - [Error Handling](#error-handling)
11. [Troubleshooting](#troubleshooting)

## Understanding Smart Grid Manager API
### SGM API Hierarchy

The Smart Grid Manager system follows a hierarchical structure:

- **Sites**: Physical locations containing energy assets (e.g., solar farms, wind parks, industrial facilities, EV charging stations)
- **Nodes**: Edge devices at a site that communicate with the central system
- **Devices**: Actual hardware (inverters, batteries, wind turbines, EV chargers, generators, etc.) connected to nodes
- **Device Types**: Templates defining properties and capabilities of different types of devices

The API communicates with edge nodes through the [Helin Platform](), which then relay commands to the physical devices using various protocols (Modbus, MQTT, etc.).

At this point a one-to-many relation exists between the layers, meaning a multiple nodes can exist at a site, but not vice-versa.

### Command Processing System

Smart Grid Manager employs control mechanisms organized into two primary categories: API setpoints and Edge Setpoints. Understanding these systems is essential for predictable control behavior.
API setpoints are the actual setpoints executed by the user, Edge setpoints are the resulting setpoints transferred to the asset by the Node on the Edge.

#### API Setpoints

Smart Grid Manager API supports multiple methods to create setpoints for assets, with a clear priority hierarchy:

1. **[Immediate setpoint (% or kW)](#direct-api-requests)** (Highest Priority)
   - Manual control requests initiated through the API endpoints
   - Immediately overrides any existing setpoints from other sources
   - Provides direct control for operators and integration systems
   - Ideal for emergency responses or manual testing scenarios

2. **[Dynamic setpoint (% and kW)]()**
   - PID controlled setpoint that corrects for local use
   - kW setting for the measurement point
   - on/off switch for the PV system

3. **[Scheduled setpoint (% or kW)](#scheduled-commands)**
   - Time-based setpoints in UTC defined in 15-minute increments
   - Automatically activated at the scheduled times
   - Only takes effect if no API request is currently active
   - Provides planned, automated control throughout the day

4. **[Fallback setpoint (% or kW)](#watchdog-mechanism)**
   - Safety mechanism that activates when higher-priority controls timeout
   - Restores devices to safe states automatically
   - Reverts to scheduled values or device defaults as appropriate
   - Ensures devices don't remain in curtailed states indefinitely

When multiple delivery methods attempt to control a device simultaneously, the system always respects this priority order, with API requests taking precedence over scheduled commands, and watchdog acting as a safety fallback.

#### Edge Setpoints

Setpoints delivered to the Edge Node are transfered to the asset on site in one of these ways:

1. **[Singular write (to be deprecated)]()**
   - Setpoint is applied once to the device
   - No automatic repetition of the command
   - Remains in effect until explicitly changed, by for example a [Watchdog control request](#watchdog-mechanism). 
   - Used for immediate, one-time value changes

2. **[Cyclic write (default)](#cyclic-writer)**
   - Setpoint is repeated at defined intervals
   - Ensures consistent device state over time
   - Protects against device memory loss or power cycles
   - Interval defined by site's `cyclic_write_timeout` setting
   - Only reinforces values, doesn't change existing setpoints
   - Operates independently on edge nodes for reliability

3. **[Cyclic control write (for dynamic setpoint)]()**
   - Setpoint is repeated at defined intervals
   - Ensures consistent device state over time
   - Operates independently on edge nodes for reliability
   - actual value updated every cycle based on the PID controller

4. **[Fallback / watchdog]()**
   - Continuously monitors actual measurements versus desired setpoints
   - Automatically calculates optimal adjustments using PID algorithms
   - Maintains desired power levels despite external variations
   - Intercepts and manages setpoints from delivery methods

The entire command processing system ensures reliable, predictable control with appropriate failsafe mechanisms and clear hierarchies for operational certainty.

### API Setpoints in detail

#### Immediate Setpoints

Direct API requests represent the highest priority control method in the Smart Grid Manager system, providing immediate manual control over grid assets.

1. **Immediate Command Execution**:
   - Commands are sent directly through the API endpoints

2. **Request Structure**:
   - Contains the target device(s) or category identification
   - Specifies the desired control value to be applied
   - May include additional parameters for specific control scenarios

3. **Implementation Details**:
   - Requests are validated for correct format and permissible values
   - Success/failure status is reported back to the API caller

4. **Timeout Behavior**:
   - Direct API requests remain in effect until explicitly changed or until timeout
   - The timeout is defined by the site's `curtailment_time_limit` parameter (typically 60-90 seconds)
   - After timeout, the watchdog mechanism activates to revert to appropriate values

Direct API requests give operators full manual control when needed, ensuring human oversight can always take precedence in the control hierarchy.

#### Scheduled Setpoints

Scheduled commands allow for time-based control (in UTC) of grid assets, enabling automated operation based on predefined time slots throughout the day.

1. **Schedule Definition**:
   - Schedules are defined for specific dates and device categories
   - Time slots are set in UTC in 15-minute increments (00:00, 00:15, 00:30, 00:45)
   - Each time slot specifies a control value to be applied

2. **Synchronization Process**:
   - Schedules are created or updated via the API
   - Schedule data is synchronized to edge nodes
   - Edge nodes store the schedules locally for reliability

3. **Time-based Activation**:
   - As each time slot is reached, the corresponding value is applied
   - Transitions happen automatically at the scheduled times
   - Only takes effect if no higher priority control is active
   - All actions performed in Device Specific Timezone.. 

4. **Priority Logic**:
   - Scheduled commands are overridden by direct API requests
   - Missing time slots default to the device's default value

5. **Advanced Features**:
   - Multi-day scheduling allowed
   - Different schedules can be applied to different device categories at a single site
   - Schedule validation ensures all values are within safe limits
   - Synchronization status tracking confirms successful deployment

Scheduled commands provide powerful automated control based on time patterns, enabling predictable operation without constant manual intervention.

#### Dynamic Setpoint

Smart Grid Manager includes a closed loop control system that enables automatic, real-time adjustments to maintain desired power setpoints. This PID-based control algorithm provides precise management of energy assets without requiring constant manual intervention.

1. **Continuous Monitoring**:
   - The system constantly reads measurements from a meter device & corrects setpoint
   - Readings that are more than 10 seconds old are considered stale and trigger fallback behavior

2. **PID Control Algorithm**:
   - Proportional-Integral-Derivative controller calculates optimal adjustments
   - Anti-windup mechanism prevents integral term from growing excessively
   - Rate limiting prevents abrupt changes that could destabilize the grid

3. **Feedback Loop**:
   - Adjustments are automatically scaled based on device capabilities
   - Commands are constrained within safe operating limits

4. **Configuration Options**:
   - Proportional and integral gain parameters control responsiveness
   - Maximum rate of change limits how quickly adjustments can occur
   - Fallback setpoint provides safe operation if readings become stale
   - Time step determines how frequently adjustments are calculated

5. **Resilience Features**:
   - Handles communication interruptions with fallback values
   - Automatically recovers from errors during measurement or control
   - Maintains setpoint even during temporary disturbances

When enabled, the closed loop controller intercepts all device control requests (except direct API calls), treats them as setpoint changes, and then continuously manages device values to maintain that setpoint.

#### Fallback setpoint
The watchdog mechanism is a critical safety feature that ensures devices revert to known safe states if communication or control failures occur.

1. **Timeout Monitoring**:
   - Tracks the time elapsed since the last direct API request
   - Compares against the site's `curtailment_time_limit` parameter (typically 60-90 seconds)
   - Activates automatically when the time limit is exceeded
   - Operates independently on each edge node

2. **Default Value Determination**:
   - Checks if a scheduled command is active for the current time
   - If a schedule exists, uses the scheduled value for the current time slot
   - If no schedule exists, reverts to the device's default value
   - Default values are predefined safe operating points for each device

3. **Safety Implementation**:
   - Runs as a high-priority process on edge nodes
   - Cannot be disabled to ensure failsafe operation
   - Executes even during communication interruptions with the central system

4. **Automatic Recovery**:
   - Prevents devices from remaining in curtailed states indefinitely
   - Creates log entries when activated to track occurrences
   - Allows normal control to resume once proper commands are received

5. **Use Cases**:
   - Network interruptions between API and edge nodes
   - Control system failures or crashes
   - Abandoned curtailment operations
   - Temporary manual curtailment that should not persist

The watchdog mechanism forms a critical safety layer in the system, ensuring that temporary control measures don't inadvertently become permanent states.

### Edge Setpoint Transfer in detail

The cyclic writer operates as the lowest priority control mechanism, providing a consistent background process that ensures device values maintain stability over time.

#### Cyclic write (default)

1. **Periodic Operation**:
   - Runs on a configurable interval defined by the site's `cyclic_write_timeout` setting
   - Typically operates every few seconds to few hours depending on configuration

2. **Value Reinforcement**:
   - Writes the appropriate value to each device periodically
   - Rewrites values so that asset can monitor Node health

3. **Value Determination**:
   - If a scheduled command is active, writes the scheduled value
   - If no schedule exists, writes the device's default value
   - Never overwrites values set by higher priority mechanisms
   - Maintains values consistent with system intent
   - value sent is automatically rate limited to match operational requirements

4. **Implementation Details**:
   - Runs on the edge nodes independently

5. **Benefits and Use Cases**:
   - Handles devices that may "forget" settings over time
   - Ensures system stability during extended operation

The cyclic writer serves as a reliability enhancement, ensuring that the intended system state persists even over extended operational periods without active intervention.

#### Rate limiting & setpoint update rate

Although Solar inverters are usually capable of high rates of change in their output, that is not always allowed by regulations and / or other hardware such as transformers.
For certain usage like trading it can be beneficial to update setpoints more often. But by updating more often the risk grows that a full on/off cycle is taking it's toll on the installed electric system.
Therfore we use a rate limiter that will limit the rate of change on the final setpoint sent to the hardware.

We follow the regulations that are designed for this purpose.
The dutch [grid regulations](https://www.netbeheernederland.nl/sites/default/files/2024-02/e02_-_20231223_-_20240116_-_netcode_elektriciteit.pdf) state two ways to set this up:
- 0 to full power in 5 minutes (20%/min) for normal use (page 32)
- faster for FFR and other balancing functionalities (up to 4s on to off)

The default setting is 20% as prescribed by the grid regulations. If you need a faster interaction, that is to be verified both legally and functionally with the party responsible for the operation of the asset.

The setting can be changed thorugh the API on a per-site basis as found in [Site Rate limiting]().




## Authentication

The Smart Grid Manager API requires two-step authentication for all endpoints:

1. An **Auth0 Bearer Token** for primary authentication
2. An **X-Management-Token** for secondary authorization and resource filtering

### Obtaining an Auth0 Bearer Token

First, you must obtain a Bearer token from Auth0:

```bash
curl -X POST "https://polarisedge.eu.auth0.com/oauth/token" \
  -H "Content-Type: application/json" \
  -d '{
    "audience": "curtailment-api",
    "grant_type": "client_credentials",
    "client_id": "your-client-id",
    "client_secret": "your-client-secret"
  }'
```

Response:
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cC...",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```

The client ID and client secret are provided by your system administrator. The access token is valid for a limited time (usually 24 hours) and will need to be refreshed periodically. Please note there's a limited amount of tokens you can request. 

### X-Management-Token

In addition to the Bearer token, you also need an X-Management-Token:

```bash
# Example authentication headers
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC...
X-Management-Token: your-management-token-here
```

There are two types of X-Management-Tokens:
- **Root Token**: Grants full access to all operations and endpoints
- **Site Management Token**: Grants access only to specified sites and their related resources

> 💡 **Tip**: To obtain a site management token, contact your system administrator.

### Managing Site Management Tokens

If you have a Root Management Token, you can create, update, and delete site management tokens. These tokens provide limited access to specific sites and their resources, allowing you to safely delegate control to other users or systems.

#### Listing All Management Tokens

To view all existing management tokens:

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/auth-tokens" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-root-management-token"
```

Response:
```json
[
  {
    "id": "e9f01a2b-3c4d-5e6f-7a8b-9c0d1e2f3a4b",
    "name": "Production Site Token",
    "token": "tok_1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d",
    "sites": [
      "01751407-d87b-4d89-b47f-cca4dfb85a92",
      "85a2dfb4-cca4-47fb-89bd-87b4d7014751"
    ]
  },
  {
    "id": "f01a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
    "name": "Test Site Token",
    "token": "tok_2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e",
    "sites": [
      "c7d8e9f0-1a2b-3c4d-5e6f-7a8b9c0d1e2f"
    ]
  }
]
```

#### Creating a New Management Token

To create a new site management token:

```bash
curl -X POST "https://smartsolar.polarisedge.com/v1/auth-tokens" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-root-management-token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "New Site Access",
    "sites": ["01751407-d87b-4d89-b47f-cca4dfb85a92"]
  }'
```

Response:
```json
{
  "id": "d8e9f01a-2b3c-4d5e-6f7a-8b9c0d1e2f3a",
  "name": "New Site Access",
  "token": "tok_3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f",
  "sites": [
    "01751407-d87b-4d89-b47f-cca4dfb85a92"
  ]
}
```

#### Updating a Management Token

To update an existing site management token (e.g., changing its name or associated sites):

```bash
curl -X PUT "https://smartsolar.polarisedge.com/v1/auth-tokens/d8e9f01a-2b3c-4d5e-6f7a-8b9c0d1e2f3a" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-root-management-token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Site Access",
    "sites": [
      "01751407-d87b-4d89-b47f-cca4dfb85a92",
      "85a2dfb4-cca4-47fb-89bd-87b4d7014751"
    ]
  }'
```

Response:
```json
{
  "id": "d8e9f01a-2b3c-4d5e-6f7a-8b9c0d1e2f3a",
  "name": "Updated Site Access",
  "token": "tok_3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f",
  "sites": [
    "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "85a2dfb4-cca4-47fb-89bd-87b4d7014751"
  ]
}
```

#### Deleting a Management Token

To delete a site management token:

```bash
curl -X DELETE "https://smartsolar.polarisedge.com/v1/auth-tokens/d8e9f01a-2b3c-4d5e-6f7a-8b9c0d1e2f3a" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-root-management-token"
```

A successful deletion will return a 204 No Content status with no response body.

> ⚠️ **Important**: When a token is deleted, any systems or users using that token will immediately lose access. Make sure to update any clients or services before deleting tokens.

### Using the Authentication in Requests

All API requests must include both authentication methods in the headers:

```bash
# Example request headers setup for cURL
curl -X GET "https://smartsolar.polarisedge.com/v1/endpoint" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token" \
  -H "Content-Type: application/json"
```

## Health and Version

### Check API Health

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/ping" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "ping": "pong!"
}
```

### Get Site Health Report

This endpoint provides a health report for all nodes across all sites, showing their connection status.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/site-health-report" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "tenant_tag": "tenant-123",
    "site_tag": "solarpanel_site_a",
    "node_name": "node-01",
    "healthy": true
  },
  {
    "tenant_tag": "tenant-123",
    "site_tag": "solarpanel_site_b",
    "node_name": "node-02",
    "healthy": false
  }
]
```

### Get API Version

This endpoint provides version information about the API and database.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/version" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "alembic_version": "ff5ac067b6b2",
  "git_version": "v1.2.3"
}
```

## Site Management

### List All Sites

Retrieve a list of all accessible sites.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "created_at": "2024-08-01T10:52:55.330076Z",
    "site_tag": "solarpanel_site_a",
    "name": "Solar Panel Site A",
    "meta_data": {},
    "curtailment_time_limit": 60
  },
  {
    "id": "85a2dfb4-cca4-47fb-89bd-87b4d7014751",
    "created_at": "2024-08-01T09:30:22.150032Z",
    "site_tag": "solarpanel_site_b",
    "name": "Solar Panel Site B",
    "meta_data": {},
    "curtailment_time_limit": 90
  }
]
```

### Get Site Details

Retrieve detailed information about a specific site.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
  "created_at": "2024-08-01T10:52:55.330076Z",
  "site_tag": "solarpanel_site_a",
  "name": "Solar Panel Site A",
  "meta_data": {},
  "curtailment_time_limit": 60,
  "metrics": [
    {
      "key": "P_Actual_kW",
      "value": 1249,
      "timestamp": "2024-08-01T10:52:55.330076Z"
    },
    {
      "key": "V_Wind_m/s",
      "value": 3.00223,
      "timestamp": "2024-08-01T10:52:55.330076Z"
    }
  ]
}
```

## Node Management

### List All Nodes

Retrieve a list of all accessible nodes across all sites.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/nodes" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
    "created_at": "2024-08-01T10:52:55.330076Z",
    "polaris_node_name": "node-01",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "meta_data": {},
    "curtail_in_batch": true,
    "iothub": "iothub-euw-polaris-engineering"
  },
  {
    "id": "2d3f4a5b-6c1e-bd8a-4f21-5e9d14b3c7a0",
    "created_at": "2024-08-01T09:30:22.150032Z",
    "polaris_node_name": "node-02",
    "site_id": "85a2dfb4-cca4-47fb-89bd-87b4d7014751",
    "meta_data": {},
    "curtail_in_batch": true,
    "iothub": "iothub-euw-polaris-engineering"
  }
]
```

### Get Node Details

Retrieve detailed information about a specific node.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/nodes/14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
  "created_at": "2024-08-01T10:52:55.330076Z",
  "polaris_node_name": "node-01",
  "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
  "meta_data": {},
  "curtail_in_batch": true,
  "iothub": "iothub-euw-polaris-engineering",
  "configuration": {
    "module": "PolarisInverterCurtailment"
  }
}
```

### List Nodes for a Site

Retrieve all nodes associated with a specific site.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/nodes" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
    "created_at": "2024-08-01T10:52:55.330076Z",
    "polaris_node_name": "node-01",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "meta_data": {},
    "curtail_in_batch": true,
    "iothub": "iothub-euw-polaris-engineering"
  }
]
```

## Device Management

### List All Devices

Retrieve a list of all accessible devices across all sites.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/devices" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "a5b6c7d8-e9f0-1a2b-3c4d-5e6f7a8b9c0d",
    "created_at": "2024-08-01T10:52:55.330076Z",
    "device_tag": "SMA_STP60_01",
    "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
    "device_type_id": "9c0d8b7a-6f5e-4d3c-2b1a-0f9e8d7c6b5a",
    "max_value": 100,
    "min_value": 0,
    "default_value": 100,
    "value": 80,
    "meta_data": {},
    "include_in_curtailment": true,
    "category": "inverter",
    "peak_power_kw": 60
  }
]
```

### Get Device Details

Retrieve detailed information about a specific device.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/devices/a5b6c7d8-e9f0-1a2b-3c4d-5e6f7a8b9c0d" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "id": "a5b6c7d8-e9f0-1a2b-3c4d-5e6f7a8b9c0d",
  "created_at": "2024-08-01T10:52:55.330076Z",
  "device_tag": "SMA_STP60_01",
  "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
  "device_type_id": "9c0d8b7a-6f5e-4d3c-2b1a-0f9e8d7c6b5a",
  "max_value": 100,
  "min_value": 0,
  "default_value": 100,
  "value": 80,
  "meta_data": {},
  "include_in_curtailment": true,
  "category": "inverter",
  "peak_power_kw": 60,
  "modbus_server_host": "192.168.1.100",
  "modbus_server_port": 502,
  "modbus_id": 1
}
```

### List Devices for a Site

Retrieve all devices associated with a specific site.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/devices" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "a5b6c7d8-e9f0-1a2b-3c4d-5e6f7a8b9c0d",
    "created_at": "2024-08-01T10:52:55.330076Z",
    "device_tag": "SMA_STP60_01",
    "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
    "device_type_id": "9c0d8b7a-6f5e-4d3c-2b1a-0f9e8d7c6b5a",
    "max_value": 100,
    "min_value": 0,
    "default_value": 100,
    "value": 80,
    "meta_data": {},
    "include_in_curtailment": true,
    "category": "inverter",
    "peak_power_kw": 60
  }
]
```

### Get Devices for a Node

Retrieve all devices associated with a specific node on a site.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/nodes/14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b/devices" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "a5b6c7d8-e9f0-1a2b-3c4d-5e6f7a8b9c0d",
    "created_at": "2024-08-01T10:52:55.330076Z",
    "device_tag": "SMA_STP60_01",
    "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
    "device_type_id": "9c0d8b7a-6f5e-4d3c-2b1a-0f9e8d7c6b5a",
    "max_value": 100,
    "min_value": 0,
    "default_value": 100,
    "value": 80,
    "meta_data": {},
    "include_in_curtailment": true,
    "category": "inverter",
    "peak_power_kw": 60
  }
]
```

### Get Device Categories for a Site

Retrieve all device categories for a specific site, with devices grouped by category.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/device-categories" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "inverter": [
    {
      "id": "a5b6c7d8-e9f0-1a2b-3c4d-5e6f7a8b9c0d",
      "created_at": "2024-08-01T10:52:55.330076Z",
      "device_tag": "SMA_STP60_01",
      "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
      "device_type_id": "9c0d8b7a-6f5e-4d3c-2b1a-0f9e8d7c6b5a",
      "max_value": 100,
      "min_value": 0,
      "default_value": 100,
      "value": 80,
      "meta_data": {},
      "include_in_curtailment": true,
      "category": "inverter",
      "peak_power_kw": 60
    }
  ],
  "battery": [
    {
      "id": "b6c7d8e9-f01a-2b3c-4d5e-6f7a8b9c0d1e",
      "created_at": "2024-08-01T10:52:55.330076Z",
      "device_tag": "IWELL_BATTERY_01",
      "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
      "device_type_id": "8b7a6f5e-4d3c-2b1a-0f9e-8d7c6b5a4b3c",
      "max_value": 100,
      "min_value": 0,
      "default_value": 100,
      "value": 100,
      "meta_data": {},
      "include_in_curtailment": true,
      "category": "battery",
      "peak_power_kw": 30
    }
  ],
  "wind_turbine": [
    {
      "id": "c7d8e9f0-1a2b-3c4d-5e6f-7a8b9c0d1e2f",
      "created_at": "2024-08-01T10:52:55.330076Z",
      "device_tag": "WIND_TURBINE_01",
      "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
      "device_type_id": "7a6f5e4d-3c2b-1a0f-9e8d-7c6b5a4b3c2d",
      "max_value": 100,
      "min_value": 0,
      "default_value": 100,
      "value": 90,
      "meta_data": {},
      "include_in_curtailment": true,
      "category": "wind_turbine",
      "peak_power_kw": 250
    }
  ],
  "ev_charger": [
    {
      "id": "d8e9f01a-2b3c-4d5e-6f7a-8b9c0d1e2f3a",
      "created_at": "2024-08-01T10:52:55.330076Z",
      "device_tag": "EV_CHARGER_01",
      "node_id": "14b3c7a0-5e9d-4f21-bd8a-6c1e2d3f4a5b",
      "device_type_id": "6f5e4d3c-2b1a-0f9e-8d7c-6b5a4b3c2d1e",
      "max_value": 100,
      "min_value": 0,
      "default_value": 80,
      "value": 60,
      "meta_data": {},
      "include_in_curtailment": true,
      "category": "ev_charger",
      "peak_power_kw": 22
    }
  ]
}
```

## Control Operations

### Control Devices by Category

Set a control value for all devices in a category at a site. This API works consistently across all asset types (solar inverters, wind turbines, batteries, EV chargers, etc.).

```bash
curl -X PUT "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/curtail-category" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token" \
  -H "Content-Type: application/json" \
  -d '{
    "value": 50,
    "category": "inverter"
  }'
```

Response:
```json
{
  "status": "success",
  "result": {
    "updated": {
      "SMA_STP60_01": null
    },
    "not_updated": {}
  }
}
```

### Control Specific Devices

Set a control value for specific devices in a category at a site. This is useful when you want to target particular assets in a more granular way.

```bash
curl -X PUT "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/curtail-category" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token" \
  -H "Content-Type: application/json" \
  -d '{
    "value": 50,
    "category": "inverter",
    "device_tags": ["SMA_STP60_01"]
  }'
```

Response:
```json
{
  "status": "success",
  "result": {
    "updated": {
      "SMA_STP60_01": null
    },
    "not_updated": {}
  }
}
```

## Logs

### Get Logs for All Sites

Retrieve curtailment logs for all sites.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/logs" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "c7d8e9f0-1a2b-3c4d-5e6f-7a8b9c0d1e2f",
    "created_at": "2024-08-01T15:22:10.330076Z",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "status": "success",
    "category": "inverter",
    "watchdog": false,
    "value": 50,
    "result": {
      "updated": {
        "SMA_STP60_01": null
      },
      "not_updated": {}
    }
  }
]
```

### Get Logs for a Specific Site

Retrieve curtailment logs for a specific site.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/logs?limit=5" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "c7d8e9f0-1a2b-3c4d-5e6f-7a8b9c0d1e2f",
    "created_at": "2024-08-01T15:22:10.330076Z",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "status": "success",
    "category": "inverter",
    "watchdog": false,
    "value": 50,
    "result": {
      "updated": {
        "SMA_STP60_01": null
      },
      "not_updated": {}
    }
  },
  {
    "id": "d8e9f01a-2b3c-4d5e-6f7a-8b9c0d1e2f3a",
    "created_at": "2024-08-01T14:15:05.120987Z",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "status": "success",
    "category": "inverter",
    "watchdog": true,
    "value": 100,
    "result": {
      "updated": {
        "SMA_STP60_01": null
      },
      "not_updated": {}
    }
  }
]
```

### Get Device Error Logs

Retrieve error logs for devices across all sites.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/device-error-logs?period=24" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "01751407-d87b-4d89-b47f-cca4dfb85a92": {
    "site_name": "Solar Panel Site A",
    "devices": {
      "SMA_STP60_01": {
        "total_errors": 2,
        "error_details": [
          {
            "timestamp": "2024-08-01T08:15:10.330076Z",
            "error": "Modbus connection timeout"
          },
          {
            "timestamp": "2024-08-01T09:22:30.120987Z",
            "error": "Invalid value range"
          }
        ]
      }
    }
  }
}
```

## Schedules

### List All Schedules

Retrieve all curtailment schedules in UTC across all sites.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/schedules" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "e9f01a2b-3c4d-5e6f-7a8b-9c0d1e2f3a4b",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "category": "inverter",
    "valid_on": "2024-08-05",
    "schedule": {
      "09:00": 50,
      "12:00": 75,
      "15:00": 60,
      "18:00": 100
    },
    "synchronization_result": null
  }
]
```

### Create a Schedule

Create a new curtailment schedule in UTC for a site.

```bash
curl -X POST "https://smartsolar.polarisedge.com/v1/schedules" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token" \
  -H "Content-Type: application/json" \
  -d '{
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "category": "inverter",
    "valid_on": "2024-08-06",
    "schedule": {
      "09:00": 50,
      "12:00": 75,
      "15:00": 60,
      "18:00": 100
    }
  }'
```

Response:
```json
{
  "id": "f01a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
  "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
  "category": "inverter",
  "valid_on": "2024-08-06",
  "schedule": {
    "09:00": 50,
    "12:00": 75,
    "15:00": 60,
    "18:00": 100
  },
  "synchronization_result": null
}
```

### Update a Schedule

Update an existing schedule in UTC for a specific date and category.

```bash
curl -X PUT "https://smartsolar.polarisedge.com/v1/schedules/2024-08-06/inverter" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token" \
  -H "Content-Type: application/json" \
  -d '{
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "schedule": {
      "09:00": 60,
      "12:00": 80,
      "15:00": 70,
      "18:00": 100
    }
  }'
```

Response:
```json
{
  "id": "f01a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
  "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
  "category": "inverter",
  "valid_on": "2024-08-06",
  "schedule": {
    "09:00": 60,
    "12:00": 80,
    "15:00": 70,
    "18:00": 100
  },
  "synchronization_result": null
}
```

### Get Schedules for a Date

Retrieve all schedules in UTC for a specific date.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/schedules/2024-08-06" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "f01a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "category": "inverter",
    "valid_on": "2024-08-06",
    "schedule": {
      "09:00": 60,
      "12:00": 80,
      "15:00": 70,
      "18:00": 100
    },
    "synchronization_result": null
  }
]
```

### Get Schedules for a Site

Retrieve all schedules in UTC for a specific site.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/schedules" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "e9f01a2b-3c4d-5e6f-7a8b-9c0d1e2f3a4b",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "category": "inverter",
    "valid_on": "2024-08-05",
    "schedule": {
      "09:00": 50,
      "12:00": 75,
      "15:00": 60,
      "18:00": 100
    },
    "synchronization_result": null
  },
  {
    "id": "f01a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "category": "inverter",
    "valid_on": "2024-08-06",
    "schedule": {
      "09:00": 60,
      "12:00": 80,
      "15:00": 70,
      "18:00": 100
    },
    "synchronization_result": null
  }
]
```

### Get Site Schedules for a Specific Date

Retrieve all schedules in UTC for a specific site on a specific date.

```bash
curl -X GET "https://smartsolar.polarisedge.com/v1/sites/01751407-d87b-4d89-b47f-cca4dfb85a92/schedules/2024-08-06" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
[
  {
    "id": "f01a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
    "site_id": "01751407-d87b-4d89-b47f-cca4dfb85a92",
    "category": "inverter",
    "valid_on": "2024-08-06",
    "schedule": {
      "09:00": 60,
      "12:00": 80,
      "15:00": 70,
      "18:00": 100
    },
    "synchronization_result": null
  }
]
```

### Delete a Schedule

Delete a specific schedule.

```bash
curl -X DELETE "https://smartsolar.polarisedge.com/v1/schedules/f01a2b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cC..." \
  -H "X-Management-Token: your-management-token"
```

Response:
```json
{
  "status": "success",
  "message": "Schedule deleted successfully"
}
```

## Best Practices

### Supported Device Types

Smart Grid Manager supports a comprehensive list of device types for control operations:

#### Solar Inverters
- **SMA**: STP60, SMA Inverter Manager IM20
- **Sungrow**: SG60KTL, SG110CX
- **Growatt**: MAX 60KTL
- **SolarEdge**: Inverters (via Modbus TCP)

#### Battery Systems
- **iWell Battery**: Modbus and MQTT protocol variants

#### Communication & Data Loggers
- **SMA**: DataManager M
- **Huawei**: SmartLogger 3000A
- **Sungrow**: Logger 1000
- **BlueLog XC**: Communication gateway

#### Protocol Support
- **Standard Protocols**: 
  - Modbus TCP/RTU
  - MQTT
  - XML-DA (OPC)
- **Generic Device Support**: 
  - ProxyDevice: For chaining devices together
  - Generic Modbus devices
  - MQTT-based devices

#### Special Purpose Devices
- **DemoDevice**: For testing and demonstration
- **XMLDADevice**: For OPC XML-DA protocol devices

Additional device types can be added upon request to meet specific project requirements. The system's modular architecture allows for rapid integration of new device types.

### Communication Protocols

The edge nodes communicate with physical devices using multiple industry-standard protocols:

- **Modbus TCP/RTU**: 
  - Industry-standard protocol widely used in energy devices
  - Supported for inverters, batteries, and control equipment
  - Both TCP (network) and RTU (serial) variants supported
  
- **MQTT**:
  - Lightweight, publish-subscribe messaging protocol
  - Used for IoT device communication (esp. iWell MQTT devices)
  - Low bandwidth, reliable for remote locations
  
- **OPC XML-DA**:
  - Industrial data exchange standard
  - Used for SCADA integration and industrial automation systems
  - Allows standardized access to real-time data
  
- **Proprietary APIs**:
  - Manufacturer-specific interfaces for specialized equipment
  - Custom implementations for specific device requirements
  - Wrapped in standardized interface for system consistency

### Control Values

- Control values vary by device type but typically use a consistent scale within each category:
  - For solar inverters and wind turbines: percentages (0-100), where 100 means "no curtailment" (full power) and 0 means "full curtailment" (no power output)
  - For battery systems: values may control charge/discharge rates or setpoints
  - For EV chargers: values may control charging rate or session parameters
  - For heat pumps and flexible loads: values may control operational modes or power limits
- Always refer to the specific device type documentation for exact value interpretations
- The API normalizes diverse control mechanisms to provide a consistent interface

### Watchdog Behavior

The system includes a watchdog mechanism:
- If a device is curtailed and no further curtailment command is received within `curtailment_time_limit` seconds, the device will automatically revert to its default value
- This ensures devices don't remain in a curtailed state indefinitely

### Schedules

Schedules follow some important rules:
- Intervals are in 15-minute increments in UTC (00:00, 00:15, 00:30, 00:45, etc.)
- A null value or missing time slot means the default value will be used
- Manual curtailment temporarily overrides schedules until the watchdog timeout
- Schedules are defined per day and per category in UTC 
- For maximum reliability, post your schedules more than 5 minutes before the start of the scheduled interval
- You can send schedules as many days before as you like
- Schedules are synchronized with the edge nodes and in case of network failure, the local schedule will be executed without interruption

### Error Handling

When working with the API, always check the response status:
- Successful operations return a 200 OK status with relevant data
- Failed operations return appropriate error codes (400, 401, 403, 404, 500) with error details
- Device-specific errors are logged and can be viewed via the device error logs endpoint

## Troubleshooting

### Common Issues

1. **Authentication Failures**
   - Ensure both your Bearer token and X-Management-Token are valid and have the appropriate permissions
   - Check that both authentication headers are correctly included in all requests
   - Bearer tokens expire - ensure you refresh your token before it expires (typically every 24 hours)
   - Invalid tokens typically return a 401 Unauthorized response
   - For site-specific operations, ensure your management token has access to that site

2. **Curtailment Commands Not Working**
   - Verify the device is included in curtailment (check include_in_curtailment flag)
   - Check the device is properly configured and connected (use the site health report endpoint)
   - Ensure the value is within the min_value and max_value range for the device
   - Check for communication errors with the device in the device error logs
   - Verify the node is online and responsive
   - If using category-based curtailment, ensure devices are assigned to the correct category

3. **Schedule Not Being Applied**
   - Confirm the schedule is for the correct date, site, and category
   - Check that the time format is correct (HH:MM in 15-minute increments)
   - Verify the schedule values are within valid ranges for the devices
   - Check the synchronization_result field for any errors in schedule application
   - Remember that direct API curtailment commands override schedules until the watchdog timeout

4. **Request Timeouts**
   - For operations that communicate with edge devices, timeouts may occur if devices are unresponsive
   - Check your network connectivity to the API server
   - For large sites with many devices, batch operations may take longer to complete
   - Consider increasing timeout settings in your API client for curtailment operations

5. **Error Response Understanding**
   - 400 Bad Request: Check your request body for formatting issues or invalid values
   - 403 Forbidden: Your token doesn't have permission for the requested resource
   - 404 Not Found: The resource (site, node, device) doesn't exist or is inaccessible
   - 409 Conflict: There's a conflict with the current state of the resource
   - 500 Server Error: Contact support with the error details from the response

### Monitoring Endpoint Health

The site health report endpoint (`/v1/site-health-report`) is valuable for monitoring the overall health of your devices and diagnosing connectivity issues. Regularly check this endpoint to proactively identify problems before they affect your operations.
