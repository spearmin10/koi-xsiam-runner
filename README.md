# koi-xsiam-runner

`koi-xsiam-runner` is an automation playbook and content bundle designed for **Cortex XSIAM**. It enables administrators to schedule and execute custom "Koi Script Packages" periodically across targeted endpoint hosts running the Cortex Agent.

---

## 🚀 Features

* **Designed to run on scheduled Jobs:** Integrates seamlessly with XSIAM Jobs to handle periodic script deployments.
* **Granular Target Filtering:** Filter target endpoints by Endpoint Groups, specific Hostnames, and Operating Systems (e.g., Windows, Linux).
* **Alert Notifications:** Automatically dispatches email notifications via designated mail instances upon execution completion.

---

## 📦 Installation

1. Download the `koi-xsiam-runner-bundle.tar.gz`.
2. Log in to your Cortex XSIAM console.
3. Navigate to **Settings** > **Server Settings** > **Custom Content** > **Upload custom content**.
4. Upload the `koi-xsiam-runner-bundle.tar.gz` file to import all components.

---

## 📂 Included Contents

This bundle contains the following Cortex XSIAM contents:

### Playbooks

* **`Koi Script Runner` (Main Playbook)**
  * The main playbook configured in Jobs to execute the Koi script package and handle notification processing based on the configuration.
* **`MMAA - Koi Script Runner`**
  * Sub-playbook utilized by the main playbook to execute the Koi script package and handle notification processing.
* **`MMAA - Run Endpoint Script`**
  * Low-level playbook responsible for dispatching the actual script commands to the target agents.

### Lists

* **`Koi Script Runner - Sample`**
  * A sample JSON list template used to define execution targets, script properties, and notification preferences.

---

## 🛠️ Usage

### Step 1: Set Up the Configuration

1. In XSIAM, navigate to **Settings** > **Configurations** > **Object Setup** > **Lists**.
2. Locate `Koi Script Runner - Sample`, clone it (or use it as a reference), and create a new list named `Koi Script Runner`.
3. Define your target script name, endpoint scopes, and notification recipient details inside the JSON format.

### Step 2: Schedule the Automation Job

1. Navigate to **Investigation & Response** > **Automation** > **Jobs**.
2. Click **+ New Job** and set the Trigger type to `Time triggered` (to run on a recurring schedule).
3. Select **`Koi Script Runner`** as the execution playbook.

---

## ⚙️ Configuration Guide (`Koi Script Runner` Lists Syntax)

This guide outlines the JSON structure, core parameters, and specific formatting of the `Koi Script Runner` required when defining configurations inside XSIAM Lists.

### 1. Basic Structure

The configuration must be formatted as a **JSON Array**, where each object represents a distinct execution job. To run multiple script packages or target groups in order, append additional objects to the array.

```json
[
    {
        "disabled": false,
        "script": {
            "name": "Koi Script Package"
        },
        "target": {
            "endpoint_groups": [
                "Koi Endpoints"
            ],
            "endpoint_hostnames": [
            ],
            "endpoint_os": "Windows"
        },
        "notification": {
            "recipients": [
                "koi@example.com"
            ]
        }
    }
]

```

---

### 2. Parameter Details

#### 2.1. Root Fields

| Field Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `disabled` | Boolean | No | `false` | If set to `true`, this entry will be skipped entirely. If omitted, it defaults to `false` (enabled). |
| `script` | Object | **Yes** | - | Contains configuration properties regarding the script file to be executed. |
| `target` | Object | **Yes** | - | Defines scope boundaries and target filtering criteria for endpoint execution. |
| `notification` | Object | No | - | Configures post-execution status reporting and alerting workflows. |

#### 2.2. `script` Object (Required)

| Field Name | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | String | **Yes** | Specifies the exact name of the target script package to execute. |

#### 2.3. `target` Object (Required)

| Field Name | Type | Required | Description |
| --- | --- | --- | --- |
| `endpoint_groups` | Array (String) | **No** | List of targeted endpoint group names. Checked as an AND condition with endpoint_hostnames. |
| `endpoint_hostnames` | Array (String) | **No** | List of specific hostnames to target. Checked as an AND condition with endpoint_groups. |
| `endpoint_os` | String | **Yes** | Restricts execution by OS type (e.g., `Windows`, `Linux` or `macOS`). Scripts will only execute on matching systems. |

> ⚠️ **Important:** Either `endpoint_groups` or `endpoint_hostnames` must have non-empty values to target valid endpoints.

#### 2.4. `notification` Object (Optional)

| Field Name | Type | Required | Description |
| --- | --- | --- | --- |
| `sendmail_instance` | Object | **No** | The configuration block for the mail sender integration that supports the `send-mail` command. |
| `sendmail_instance.name` | String | **No** | The name of an active integration instance. |
| `recipients` | Array (String) | **No** | A list of destination email addresses for administrative alerts or operational routing. |

---

### 3. Advanced Examples

#### Multi-Target / Parallel Job Execution

If you need to deploy different scripts to different operating systems or push notifications to separate mailing lists, extend the array as follows:

```json
[
    {
        "script": {
            "name": "Koi Script Package - Windows"
        },
        "target": {
            "endpoint_groups": [
                "Koi Endpoints"
            ],
            "endpoint_hostnames": [
            ],
            "endpoint_os": "Windows"
        },
        "notification": {
            "sendmail_instance": {
                "name": "internal-smtp"
            },
            "recipients": [
                "koi-win@example.com"
            ]
        }
    },
    {
        "script": {
          "name": "Koi Script Package - macOS"
        },
        "target": {
            "endpoint_groups": [
                "Koi Endpoints"
            ],
            "endpoint_hostnames": [
            ],
            "endpoint_os": "macOS"
        },
        "notification": {
            "sendmail_instance": {
                "name": "internal-smtp"
            },
            "recipients": [
                "koi-mac@example.com"
            ]
        }
    }
]

```
