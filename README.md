# bank-system-
🏦 Simple Bank Account in Python A beginner-friendly Python project that simulates basic bank account operations like deposit, withdrawal, and balance inquiry. This project is ideal for learning object-oriented programming (OOP) concepts and practicing Python fundamentals.


import json
import os
from getpass import getpass

DATA_FILE = "bank_data.json"


# ---------------------------
# Load & Save Data
# ---------------------------
def load_data():
    if not os.path.exists(DATA_FILE):
        return {}  # empty dict if no file
    try:
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    except json.JSONDecodeError:
        # If file gets corrupted, reset it
        return {}


def save_data(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f, indent=4)


# ---------------------------
# Bank Account Class
# ---------------------------
class BankAccount:
    def _init_(self, account_number, pin, balance=0, history=None):
        self.account_number = account_number
        self.pin = pin
        self.balance = balance
        self.history = history if history is not None else []

    def verify_pin(self):
        attempts = 3
        while attempts > 0:
            try:
                entered_pin = getpass("Enter PIN: ")
            except:
                # fallback for terminals that don't support getpass
                entered_pin = input("Enter PIN: ")

            if entered_pin == self.pin:
                print("✔ Login successful!\n")
                return True

            attempts -= 1
            print(f"❌ Wrong PIN! Attempts left: {attempts}")

        print("⚠ Too many attempts. Account locked!")
        return False

    def deposit(self, amount):
        self.balance += amount
        self.history.append(f"Deposited ₹{amount}")
        print(f"✔ ₹{amount} deposited successfully!")

    def withdraw(self, amount):
        if amount > self.balance:
            print("❌ Not enough balance!")
            return

        self.balance -= amount
        self.history.append(f"Withdrew ₹{amount}")
        print(f"✔ ₹{amount} withdrawn successfully!")

    def change_pin(self):
        try:
            old_pin = getpass("Enter old PIN: ")
        except:
            old_pin = input("Enter old PIN: ")

        if old_pin != self.pin:
            print("❌ Incorrect old PIN!")
            return

        try:
            new_pin = getpass("Enter new PIN: ")
            confirm = getpass("Confirm new PIN: ")
        except:
            new_pin = input("Enter new PIN: ")
            confirm = input("Confirm new PIN: ")

        if new_pin == confirm:
            self.pin = new_pin
            print("✔ PIN changed successfully!")
        else:
            print("❌ PINs do not match!")

    def show_history(self):
        print("\n--- Transaction History ---")
        if not self.history:
            print("No transactions yet.")
            return
        for entry in self.history:
            print(entry)


# ---------------------------
# Create New Account
# ---------------------------
def create_account(data):
    acc_no = input("Enter new account number: ")

    if acc_no in data:
        print("❌ Account already exists!")
        return

    try:
        pin = getpass("Set PIN: ")
    except:
        pin = input("Set PIN: ")

    data[acc_no] = {
        "pin": pin,
        "balance": 0,
        "history": []
    }

    save_data(data)
    print("✔ Account created successfully!")


# ---------------------------
# Login System
# ---------------------------
def login(data):
    acc_no = input("Enter account number: ")

    if acc_no not in data:
        print("❌ Account not found!")
        return None

    account_data = data[acc_no]

    # create BankAccount object
    account = BankAccount(
        acc_no,
        account_data.get("pin", ""),
        account_data.get("balance", 0),
        account_data.get("history", [])
    )

    if account.verify_pin():
        return account

    return None


# ---------------------------
# Menu After Login
# ---------------------------
def account_menu(account, data):
    while True:
        print("\n--- ACCOUNT MENU ---")
        print("1. Deposit")
        print("2. Withdraw")
        print("3. Check Balance")
        print("4. Transaction History")
        print("5. Change PIN")
        print("6. Logout")

        choice = input("Enter choice: ")

        if choice == "1":
            try:
                amt = float(input("Enter amount to deposit: "))
                if amt > 0:
                    account.deposit(amt)
                else:
                    print("❌ Enter a positive amount.")
            except:
                print("❌ Invalid amount!")

        elif choice == "2":
            try:
                amt = float(input("Enter amount to withdraw: "))
                if amt > 0:
                    account.withdraw(amt)
                else:
                    print("❌ Enter a positive amount.")
            except:
                print("❌ Invalid amount!")

        elif choice == "3":
            print(f"💰 Current Balance: ₹{account.balance}")

        elif choice == "4":
            account.show_history()

        elif choice == "5":
            account.change_pin()

        elif choice == "6":
            print("✔ Logged out successfully!")
            break

        else:
            print("❌ Invalid option! Try again.")

        # save account changes
        data[account.account_number] = {
            "pin": account.pin,
            "balance": account.balance,
            "history": account.history
        }
        save_data(data)


# ---------------------------
# MAIN PROGRAM
# ---------------------------
def main():
    while True:
        data = load_data()

        print("\n===== WELCOME TO PYBANK =====")
        print("1. Create Account")
        print("2. Login")
        print("3. Exit")

        option = input("Choose an option: ")

        if option == "1":
            create_account(data)

        elif option == "2":
            account = login(data)
            if account:
                account_menu(account, data)

        elif option == "3":
            print("Thank you for using PyBank!")
            break

        else:
            print("❌ Invalid choice! Try again.")


# Run program
main()
