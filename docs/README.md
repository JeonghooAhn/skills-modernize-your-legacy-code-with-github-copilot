# COBOL Account Management Program

This directory documents the COBOL account-management example in `src/cobol`.
The program provides a menu to view one account balance, credit the account,
debit the account, or exit. Although this may be used as a student-account
exercise, the current implementation does not store student names, IDs, or
separate balances for multiple students.

## COBOL files

### `src/cobol/main.cob` — `MainProgram`

Entry point and user interface. `MAIN-LOGIC` repeatedly displays the menu,
accepts a numeric choice, and dispatches requests to `Operations`:

- `1` sends `TOTAL ` to display the balance.
- `2` sends `CREDIT` to add funds.
- `3` sends `DEBIT ` to subtract funds.
- `4` exits the loop and stops the program.
- Other choices display an invalid-choice message and return to the menu.

### `src/cobol/operations.cob` — `Operations`

Handles account actions based on the operation passed by `MainProgram`.

- `TOTAL ` requests the balance from `DataProgram` and displays it.
- `CREDIT` accepts an amount, reads the current balance, adds the amount,
  writes the new balance, and displays it.
- `DEBIT ` accepts an amount, reads the current balance, and subtracts and
  saves the amount only when the balance is greater than or equal to it.
  Otherwise it displays an insufficient-funds message and leaves the balance
  unchanged.

### `src/cobol/data.cob` — `DataProgram`

Stores and retrieves the account balance. The `READ` operation copies its
`STORAGE-BALANCE` to the caller's balance field; `WRITE` replaces
`STORAGE-BALANCE` with the supplied value. The starting balance is `1000.00`.
This storage is in working memory, so changes are lost when the program exits.

## Student-account business rules and behavior

- The program maintains one shared account balance; there are no student
  profiles, account selection, transaction history, or persistent records.
- A new run starts with a balance of `1000.00`.
- Credits add the entered amount without a maximum-balance check.
- A debit is accepted when the current balance is at least the entered amount.
  A debit that exceeds the balance is rejected without changing the balance;
  overdrafts are not allowed by this check.
- Amount fields use `PIC 9(6)V99`, representing up to six integer digits and
  two decimal places. The implied decimal point is not displayed by these
  fields.
- The code does not validate that entered amounts are positive or otherwise
  valid business amounts. It also does not define student-specific fees,
  minimum balances, or other student-account policies.

## Application data flow

```mermaid
sequenceDiagram
    actor User
    participant Main as MainProgram<br/>(main.cob)
    participant Ops as Operations<br/>(operations.cob)
    participant Data as DataProgram<br/>(data.cob)

    loop Until the user chooses 4
        Main->>User: Display menu
        User->>Main: Enter choice
        alt Choice 1: view balance
            Main->>Ops: TOTAL
            Ops->>Data: READ, balance
            Data-->>Ops: Current balance
            Ops-->>User: Display current balance
        else Choice 2: credit
            Main->>Ops: CREDIT
            Ops->>User: Prompt for credit amount
            User-->>Ops: Enter amount
            Ops->>Data: READ, balance
            Data-->>Ops: Current balance
            Ops->>Ops: Add amount to balance
            Ops->>Data: WRITE, updated balance
            Data-->>Ops: Store updated balance
            Ops-->>User: Display credited amount and new balance
        else Choice 3: debit
            Main->>Ops: DEBIT
            Ops->>User: Prompt for debit amount
            User-->>Ops: Enter amount
            Ops->>Data: READ, balance
            Data-->>Ops: Current balance
            alt Balance is at least the debit amount
                Ops->>Ops: Subtract amount from balance
                Ops->>Data: WRITE, updated balance
                Data-->>Ops: Store updated balance
                Ops-->>User: Display debited amount and new balance
            else Insufficient funds
                Ops-->>User: Display insufficient-funds message
            end
        else Choice 4: exit
            Main->>Main: End menu loop
        else Invalid choice
            Main-->>User: Display invalid-choice message
        end
    end
    Main-->>User: Display goodbye message
```
