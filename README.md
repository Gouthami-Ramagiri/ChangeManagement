# ServiceNow Change Management Customization

## Project Overview

This project focuses on customizing the ServiceNow Change Management process by implementing different business requirements for Change Requests.

The project demonstrates how ServiceNow can be customized using:

- Change Models
- Client Scripts
- UI Policies
- UI Actions
- Business Rules
- `g_form`
- `gs.getUserDisplayName()`

The main objective is to automate Change Request behavior and improve the user experience while creating and managing change requests.

---

## Objectives

The project implements the following Change Management requirements:

1. Display Change Models when creating a new Change Request.
2. Change the Number field's background color based on the Change Request state.
3. Flash the Short Description field when the Impact is High.
4. Make the Assigned to field visible and mandatory when an Assignment Group is selected.
5. Make the Description field mandatory when Short Description is populated.
6. Create a Save & Stay button on the Change Request form.
7. Automatically populate Work Notes with the logged-in user's name when the request reaches Scheduled state.
8. Automatically populate Description with relevant information when the request enters Review state.

---

## Change Request Flow

### Default Flow

Change  
→ Create New  
→ Change Request Form

### Customized Flow

Change  
→ Create New  
→ Select Change Model  
→ Change Request Form

A Change Model acts as a reusable configuration/template for creating Change Requests.

It can help avoid manually configuring properties such as:

- Change Type
- Risk
- Approval
- Closure
- Other Change Request configurations

---

## Technologies and ServiceNow Features

| Technology / Feature | Usage |
|---|---|
| ServiceNow | Platform used for the project |
| Change Management | Main application/module |
| Change Models | Reusable Change Request configurations |
| Client Scripts | Client-side form behavior |
| UI Policies | Dynamic field visibility and mandatory behavior |
| UI Actions | Custom buttons/actions |
| Business Rules | Server-side automation |
| JavaScript | Scripting language |
| `g_form` | Client-side form manipulation |
| `gs` | Server-side ServiceNow API |

---

# Requirements Implemented

## Requirement 1 — Change Models

### Requirement

Whenever the user selects Create New Change Request, available Change Models should be displayed.

### Implementation

The Change Request creation flow is changed to:

Change → Create New → Select Change Model → Change Request Form

Three Change Models were created:

- Normal Infrastructure Change
- Emergency Production Change
- Standard Software Change

The Change Model provides a reusable configuration for creating Change Requests.

---

## Requirement 2 — Change Number Background Color

### Requirement

Whenever the State field changes, the background color of the Number field should change according to the selected state.

### Implementation

An `onChange` Client Script is used.

| State Value | Number Field Color |
|---|---|
| `-5` | Red |
| `-4` | Blue |
| `-3` | Yellow |
| `-2` | Orange |
| `-1` | Purple |
| `3` | Gray |

Example:

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {

    if (isLoading || newValue === '') {
        return;
    }

    var numberField = g_form.getControl('number');

    if (newValue == '-5') {
        numberField.style.backgroundColor = 'red';
    } 
    else if (newValue == '-4') {
        numberField.style.backgroundColor = 'blue';
    } 
    else if (newValue == '-3') {
        numberField.style.backgroundColor = 'yellow';
    } 
    else if (newValue == '-2') {
        numberField.style.backgroundColor = 'orange';
    } 
    else if (newValue == '-1') {
        numberField.style.backgroundColor = 'purple';
    } 
    else if (newValue == '3') {
        numberField.style.backgroundColor = 'gray';
    }
}
```
## Requirement 3 — Flash Short Description for High Impact
### Requirement

Whenever the Impact is High, the Short Description field should flash three times.

### Implementation

An onChange Client Script is used.

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {

    if (newValue == '') {
        return;
    }

    if (newValue == 1) {
        g_form.flash('short_description', '#FF0000', -2);
    }
}
```
### Important Note

The script and the table must reference the same table. If the table ID is different, the script will not work correctly.

## Requirement 4 — Assignment Group and Assigned To
### Requirement

When the Assignment Group is changed:

* Assigned to should become visible.
* Assigned to should become mandatory.
### Implementation

This requirement is implemented using a UI Policy.

The UI Policy Action is configured as:
```
Field: Assigned to
Visible: True
Mandatory: True
```
## Requirement 5 — Make Description Mandatory
### Requirement

When Short Description is populated, the Description field should become mandatory.

### Implementation

A UI Policy is used.

#### Condition:

```
Short Description is not empty
```
#### UI Policy Action:
```
Description → Mandatory
```
This ensures that users provide additional information whenever a Short Description has been entered.

## Requirement 6 — Save & Stay Button
### Requirement

Create a button on the Change Request form that saves the record while keeping the user on the same page.

### Implementation

A UI Action is created as a Form Button.

The action uses:
```javascript
function saveAndStay() {
    g_form.save();
}
```
Expected Behavior

User clicks Save & Stay\
→ Record is saved\
→ User remains on the form

## Requirement 7 — Populate Work Notes in Scheduled State
### Requirement

When a Change Request enters the Scheduled state, the Work Notes field should automatically contain the logged-in user's name.

#### Example:

The current logged-in user is Rakesh
### Implementation

A Before Update Business Rule is used.
```javascript
(function executeRule(current, previous /*null when async*/) {

    if (current.state == '-2') {
        current.work_notes =
            "The current logged-in user is " +
            gs.getUserDisplayName();
    }

})(current, previous);
```
## Requirement 8 — Populate Description in Review State
### Requirement

When the Change Request enters the Review state, the Description should contain:

* Current logged-in user
* Assignment group
* State
* Current short description
## Implementation

A Before Update Business Rule is used.

```javascript
(function executeRule(current, previous) {

    if (current.state.changesTo('0')) {

        current.description =
            "Current logged-in user is " +
            gs.getUserDisplayName() +
            "\nAssignment group is " +
            current.assignment_group.getDisplayValue() +
            "\nState is: " +
            current.state.getDisplayValue() +
            "\nCurrent short description is: " +
            current.short_description;

    }

})(current, previous);
```
ServiceNow Components Used

The project uses the following ServiceNow components:
```text
Change Management
        |
        v
Change Request
        |
        +-------------------+
        |                   |
        v                   v
Change Models        Client Scripts
                            |
                            v
                       UI Policies
                            |
                            v
                       UI Actions
                            |
                            v
                     Business Rules
                            |
                            v
                  Automated Behavior
```
# Testing

The implemented functionality can be tested through the Change Request module.

## Change Models

Navigate to:
```
Change → All → New
```
Verify that the configured Change Models are displayed.

### State-Based Number Color
* Open a Change Request.
* Change the State.
* Verify that the Number field background changes according to the configured state.
### High Impact
* Set Impact to High.
* Verify that Short Description flashes.
### Assignment Group
* Change Assignment Group.
* Verify that Assigned to becomes visible.
* Verify that Assigned to becomes mandatory.
### Short Description
* Enter a Short Description.
* Verify that Description becomes mandatory.
### Save & Stay
* Modify a Change Request.
* Click Save & Stay.
* Verify that the record is saved.
* Verify that the user remains on the form.
### Scheduled State
* Move the Change Request to Scheduled.
* Save or update the record.
* Verify that Work Notes contain the logged-in user's name.
### Review State
* Move the Change Request to Review.
* Save or update the record.
* Verify that Description contains:
  1. Logged-in user
  2. Assignment group
  3. State
  4. Short description
## Key ServiceNow Concepts Learned
### Client-Side
* Client Scripts
* `g_form`
* `onChange`
* Dynamic field manipulation
* Form field styling
### Configuration
* Change Models
* UI Policies
* UI Policy Actions
### Server-Side
* Business Rules
* current
* previous
* `gs.getUserDisplayName()`
* Server-side field updates
### UI Customization
* UI Actions
* Form buttons
* Custom Save behavior
## Learning Outcome

This project provides practical experience in customizing the ServiceNow Change Management lifecycle.

It demonstrates how business requirements can be converted into ServiceNow configurations and scripts:
```
Business Requirement
        ↓
ServiceNow Configuration
        ↓
Client / Server Script
        ↓
Testing
        ↓
Automated Change Request Behavior
```
