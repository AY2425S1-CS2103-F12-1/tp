---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# InSUREance Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

ChatGPT was heavily used to help create test case templates, although the ideas behind the test cases were mostly thought by the developers.

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `ClientListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Client` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a client).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Client` objects (which are contained in a `UniqueClientList` object).
* stores the currently 'selected' `Client` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Client>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Client` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Client` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## Future Features

### Being able to add new Insurance Plan Types
As of now, we know that it can be cumbersome for users to wait for updates for new types of insurance plans to be added by us the developers. Thus, we have planned in a future update, we will allow users to create their own Insurance Plan Types to add to their clients. Do note that as these plans are created by users and not validated by us, we will not be held liable for any unintended record-keeping issues such as claim ID not being validated correctly or the claim amount not being calculated correctly!


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**: **Insurance Agents**

* has a need to manage a significant number of insurance clients
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: Our app helps insurance agents effortlessly manage new and existing clients, offering features for tracking insurance plans and claims, all while staying organised and efficient."


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​           | I want to …​                                                        | So that I can…​                                                               |
|----------|-------------------|---------------------------------------------------------------------|-------------------------------------------------------------------------------|
| `* * *`  | new user          | view the help page                                                  | refer to instructions when I forget how to use the app                        |
| `* * *`  | basic user        | add a client                                                        | keep track of client's contact details                                        |
| `* * *`  | basic user        | delete a client                                                     | remove client's contact details that I no longer need                         |
| `* *`    | intermediate user | edit a client                                                       | update client's contact when necessary                                        |
| `* * *`  | basic user        | list all clients                                                    | view all clients in the app                                                   |
| `* * *`  | efficient user    | find a client by name                                               | quickly retrieve client's contact without looking through all my contacts     |
| `* * *`  | basic user        | add an insurance plan to a client                                   | keep track of the insurance plan the client has purchased                     |
| `* * *`  | basic user        | delete an insurance plan from a client                              | remove an insurance plan that was previously purchased by the client          |
| `* *`    | intermediate user | add a claim to an insurance plan purchased by a client              | keep track of a claim filed against an insurance plan                         |
| `* *`    | intermediate user | delete a claim from an insurance plan purchased by a client         | remove a wrongly-added claim from an insurance plan                           |
| `* *`    | intermediate user | close a claim filed against an insurance plan purchased by a client | mark the claim as closed                                                      |
| `* *`    | intermediate user | list all claims of a client                                         | view all claims filed for the various insurance plans purchased by the client |
| `*`      | frequent user     | customise the theme of the app                                      | change the theme according to my mood and liking                              |
| `* *`    | basic user        | clear all contacts from the app                                     | quickly reset my app to an empty state                                        |
| `* * *`  | basic user        | exit the app                                                        | close the app when I am done using it                                         |

### Use cases

(For all use cases below, the **System** is the `InSUREance` and the **Actor** is the `user`, unless specified otherwise)

---
**Use case 01: Add a client**

**MSS**

1. User requests to add a new client
2. System checks new client details.
3. System assigns client an ID.
4. System shows successfully added a new client message.

Use case ends.

**Extensions**

* 2a. Client details are invalid.
    * 2a1. System shows invalid client details error message.
    * 2a2. User enters new client details.

    Steps 2a1-2a2 are repeated until the data entered is correct

    Use case resumes from step 3.

* 2b. Client name is identical to another client that already exists inside the system.
    * 2b1. System sends a warning about identical client to user.

    Use case resumes from step 3.
---
<a name = "UC02"></a> **Use case 02: Select a client**

**MSS**

1. User requests to select a client.
2. System checks if the index is valid.
3. System selects the client.

Use case ends.

**Extensions**

* 2a. Client index is invalid.
    * 2a1. System shows invalid index error message.
    * 2a2. User enters new index.

    Steps 2a1-2a2 are repeated until the index is valid.

    Use case resumes from step 3.
---
**Use case 03: Delete a client**

**MSS**

1.  User requests to list clients.
2.  System shows a list of clients.
3.  User [select a client](#UC02) to delete.
4.  System deletes the client.

Use case ends.

**Extensions**

* 2a. The list is empty.

Use case ends.

---
<a name = "UC04"></a>**Use case 04: Select an insurance plan**

**MSS**

1.  User selects an insurance plan ID.
2.  System checks if the ID is valid.
3.  System has selected the insurance plan ID.

Use case ends.

**Extensions**

* 2b. The insurance plan id is invalid.
    * 2b1. System shows an error message to user.

    Use case ends.

* 3a. The client does not have the specified insurance plan.
    * 3a1. System shows an error message to user.

    Use case ends.

---
**Use case 05: Add an insurance plan to a client**

**MSS**

1.  User requests to list clients.
2.  System shows a list of clients.
3.  User [select a client](#UC02) and [select an insurance plan](#UC04) to be added to the specified client.
4.  System adds the insurance plan to the client.

Use case ends.

**Extensions**

* 3a. The client already has the given insurance plan.
    * 3a1. System shows an error message.

    Use case ends.
---
**Use case 06: Remove an insurance plan from a client**

**MSS**

1.  User requests to list clients.
2.  System shows a list of clients.
3.  User [select a client](#UC02), [select an insurance plan](#UC04) of that client and requests it to be deleted.
4.  System removes the insurance plan from the client.

Use case ends.

**Extensions**

* 3a. The client does not have the specified insurance plan.
    * 3a1. System shows an error message to user.

    Use case ends.
---
**Use case 07: View all claims of a client**

**MSS**

1.  User [select a client](#UC02) and requests to view all claims of that client.
2.  System shows all claims for the client.

Use case ends.

**Extensions**

* 2a. There are no claims for the client.
    * 2a1. An appropriate message about the client having no claims will be shown.

    Use case ends.
---
<a name = "UC08"></a>**Use case 08: Select a claim of a client**

**MSS**

1.  User [select a client](#UC02), [select an insurance plan](#UC04) of the client and requests to select a claim of that specific insurance plan.
2.  System selects the claim specified by the user.

Use case ends.

**Extensions**

* 1a. Claim ID is invalid.
    * 1a1. An appropriate error message will be shown.

    Use case ends.
---
**Use case 09: Add a claim to a client**

**MSS**

1.  User [select a client](#UC02) and [select an insurance plan](#UC04) of that client and requests to add a claim to selected insurance plan.
2.  System adds the claim to the insurance plan.

Use case ends.

**Extensions**

* 2a. The claim already exists for the client.
    * 2a1. System shows an error message.

    Use case ends.
---
**Use case 09: Close a claim for a client**

**MSS**

1.  User [select a client](#UC02), [select a claim](#UC08) and requests to close that claim for the specified client.
2.  System closes the claim for the client.

Use case ends.

**Extensions**

* 3a. The claim has already been closed for the client.
    * 3a1. System shows an error message.

    Use case ends.
---

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  The data should be stored locally in a JSON file.
3.  The data should be stored in a way that complies with the Personal Data Protection Act, Singapore [[PDPA](https://www.pdpc.gov.sg/overview-of-pdpa/the-legislation/personal-data-protection-act)].
4.  Should be able to hold up to 1000 clients without a noticeable sluggishness in performance for typical usage.
5.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
6.  The app should display results within 2 seconds when searching for a client.
7.  The app should save the data entered even if it exits unexpectedly while running.
8.  The app should have a log of user actions for debugging purposes.
9.  The app should help users validate their inputs for their client's claim information as they are adding or deleting the inputs based on known market standards.

*{More to be added}*

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Insurance Agent**: The user of the app.
* **Client**: A potential customer who is keen on purchasing an insurance plan or a customer who has purchased at least one insurance plan from the insurance agent.
* **Insurance Plan ID**: A unique ID assigned to the insurance plan by the system.
* **Valid Insurance Plan ID**: An insurance plan ID that exists in the system.
* **Claim**: A formal request by a client for reimbursement for losses that are covered by specific insurance plans.
<!-- (the above definition was obtained from: https://www.iciciprulife.com/insurance-claim.html) -->
* **Claim ID**: A formal identification number provided by the Insurance Provider when an insurance agent submits a claim on behalf of the client.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a client

1. Deleting a client while all clients are being shown

   1. Prerequisites: List all clients using the `list` command. Multiple clients in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No client is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
