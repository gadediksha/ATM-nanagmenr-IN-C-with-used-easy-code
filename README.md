//ATM-nanagmenr-IN-C-with-used-easy-code

#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAX_TX 5     // number of transactions stored in mini-statement
#define INITIAL_PIN 1234
#define INITIAL_BALANCE 10000.0

typedef struct {
    int type;           // 0 = deposit, 1 = withdrawal
    double amount;
} Transaction;

void clear_input_buffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) {}
}

void print_menu() {
    printf("\n====== ATM MENU ======\n");
    printf("1. Check Balance\n");
    printf("2. Withdraw\n");
    printf("3. Deposit\n");
    printf("4. Mini-statement (last %d)\n", MAX_TX);
    printf("5. Change PIN\n");
    printf("6. Exit\n");
    printf("======================\n");
    printf("Enter choice: ");
}

void add_transaction(Transaction txs[], int *count, Transaction t) {
    // keep most recent at the end; if full, shift left
    if (*count < MAX_TX) {
        txs[*count] = t;
        (*count)++;
    } else {
        // shift left and add at end
        for (int i = 0; i < MAX_TX - 1; i++) txs[i] = txs[i + 1];
        txs[MAX_TX - 1] = t;
    }
}

int authenticate(int stored_pin) {
    int tries = 0;
    int pin;
    while (tries < 3) {
        printf("Enter your 4-digit PIN: ");
        if (scanf("%d", &pin) != 1) {
            printf("Invalid input. Try again.\n");
            clear_input_buffer();
            tries++;
            continue;
        }
        if (pin == stored_pin) {
            clear_input_buffer();
            return 1; // success
        } else {
            printf("Incorrect PIN. %d attempt(s) left.\n", 2 - tries);
            tries++;
        }
    }
    clear_input_buffer();
    return 0; // failed after 3 tries
}

int main() {
    double balance = INITIAL_BALANCE;
    int pin = INITIAL_PIN;
    Transaction txs[MAX_TX];
    int tx_count = 0;
    int choice;
    double amt;
    printf("Welcome to Simple ATM (Demo)\n");
    printf("Default PIN = %d, Default balance = %.2f\n\n", INITIAL_PIN, INITIAL_BALANCE);

    if (!authenticate(pin)) {
        printf("Too many failed attempts. Card blocked. Exiting.\n");
        return 0;
    }

    while (1) {
        print_menu();
        if (scanf("%d", &choice) != 1) {
            printf("Invalid choice. Please enter a number 1-6.\n");
            clear_input_buffer();
            continue;
        }
        clear_input_buffer();

        switch (choice) {
            case 1: // Check balance
                printf("Your current balance is: ₹%.2f\n", balance);
                break;

            case 2: // Withdraw
                printf("Enter amount to withdraw: ₹");
                if (scanf("%lf", &amt) != 1) {
                    printf("Invalid amount.\n");
                    clear_input_buffer();
                    break;
                }
                clear_input_buffer();
                if (amt <= 0) {
                    printf("Enter an amount greater than 0.\n");
                } else if (amt > balance) {
                    printf("Insufficient balance. Your balance is ₹%.2f\n", balance);
                } else {
                    balance -= amt;
                    Transaction t = {1, amt};
                    add_transaction(txs, &tx_count, t);
                    printf("Please collect your cash: ₹%.2f\n", amt);
                    printf("Remaining balance: ₹%.2f\n", balance);
                }
                break;

            case 3: // Deposit
                printf("Enter amount to deposit: ₹");
                if (scanf("%lf", &amt) != 1) {
                    printf("Invalid amount.\n");
                    clear_input_buffer();
                    break;
                }
                clear_input_buffer();
                if (amt <= 0) {
                    printf("Enter an amount greater than 0.\n");
                } else {
                    balance += amt;
                    Transaction t = {0, amt};
                    add_transaction(txs, &tx_count, t);
                    printf("₹%.2f deposited successfully.\n", amt);
                    printf("New balance: ₹%.2f\n", balance);
                }
                break;

            case 4: // Mini-statement
                if (tx_count == 0) {
                    printf("No transactions yet.\n");
                } else {
                    printf("Last %d transaction(s):\n", tx_count);
                    for (int i = 0; i < tx_count; i++) {
                        int idx = i;
                        printf("%d. %s ₹%.2f\n", i+1,
                               (txs[idx].type == 0) ? "Deposit " : "Withdraw",
                               txs[idx].amount);
                    }
                }
                printf("Available balance: ₹%.2f\n", balance);
                break;

            case 5: // Change PIN
                {
                    int oldpin, newpin, newpin2;
                    printf("Enter current PIN: ");
                    if (scanf("%d", &oldpin) != 1) {
                        printf("Invalid input.\n");
                        clear_input_buffer();
                        break;
                    }
                    clear_input_buffer();
                    if (oldpin != pin) {
                        printf("Incorrect current PIN.\n");
                        break;
                    }
                    printf("Enter new 4-digit PIN: ");
                    if (scanf("%d", &newpin) != 1) {
                        printf("Invalid input.\n");
                        clear_input_buffer();
                        break;
                    }
                    clear_input_buffer();
                    printf("Re-enter new PIN: ");
                    if (scanf("%d", &newpin2) != 1) {
                        printf("Invalid input.\n");
                        clear_input_buffer();
                        break;
                    }
                    clear_input_buffer();
                    if (newpin != newpin2) {
                        printf("PINs do not match. PIN not changed.\n");
                    } else {
                        pin = newpin;
                        printf("PIN changed successfully.\n");
                    }
                }
                break;

            case 6: // Exit
                printf("Thank you for using Simple ATM. Goodbye!\n");
                return 0;

            default:
                printf("Invalid choice. Please select 1-6.\n");
        } // switch
    } // while

    return 0;
}
