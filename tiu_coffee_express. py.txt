import datetime
import random
MENU = {
    "1": {"item": "Espresso", "price": 2.50},
    "2": {"item": "Latte", "price": 4.00},
    "3": {"item": "Cappuccino", "price": 4.50},
    "4": {"item": "Muffin", "price": 3.00},
    "5": {"item": "Croissant", "price": 2.00}
}
PROMO_CODE = "TIUSTUDENT"
DISCOUNT_RATE = 0.15
class CoffeeOrder:
    def __init__(self, customer_name):
        self.customer_name = customer_name
        self.items = []
        self.subtotal = 0.0
        self.discount_applied = 0.0
        self.final_total = 0.0
        self.receipt_id = random.randint(1000, 9999)

    def add_item(self, item_code, quantity):
        item_data = MENU.get(item_code)

        if item_data:
            price = item_data["price"]
            item_name = item_data["item"]
            line_total = price * quantity

            self.items.append({
                "name": item_name,
                "qty": quantity,
                "price": price,
                "line_total": line_total
            })

            self.subtotal += line_total
            print(f"Added {quantity}x {item_name} to the order. Current subtotal: ${self.subtotal:.2f}")
        else:
            print("Invalid item code. Please choose from the menu.")

    def apply_discount(self, user_code):
        if self.subtotal == 0:
            print("Cannot apply discount to an empty order.")
            return

        if user_code == PROMO_CODE:
            discount_amount = self.subtotal * DISCOUNT_RATE
            self.discount_applied = discount_amount
            print(f"\n SUCCESS: Discount {PROMO_CODE} applied! You saved ${discount_amount:.2f}.")
        else:
            print("\n Invalid promo code. No discount applied.")

    def calculate_total(self):
        self.final_total = self.subtotal - self.discount_applied
        return self.final_total

    def generate_receipt(self):
        if not self.items:
            return "--- Order is Empty ---"

        now = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        receipt = "=" * 40 + "\n"
        receipt += "          TIU COFFEE EXPRESS\n"
        receipt += "=" * 40 + "\n"
        receipt += f"Receipt ID: {self.receipt_id} | Date: {now}\n"
        receipt += f"Customer: {self.customer_name}\n"
        receipt += "-" * 40 + "\n"
        receipt += f"{'Item':<20}{'Qty':>5}{'Price':>8}\n"
        receipt += "-" * 40 + "\n"

        for item in self.items:

            receipt += f"{item['name']:<20}{item['qty']:>5}{item['line_total']:>8.2f}\n"

        receipt += "-" * 40 + "\n"
        receipt += f"{'SUBTOTAL:':<30}${self.subtotal:>8.2f}\n"

        if self.discount_applied > 0:
            receipt += f"{'DISCOUNT APPLIED:':<30}-${self.discount_applied:>7.2f}\n"
            receipt += "=" * 40 + "\n"

        receipt += f"{'TOTAL DUE:':<30}${self.final_total:>8.2f}\n"
        receipt += "=" * 40 + "\n"
        receipt += "  Thank you for supporting TIU events!\n"
        receipt += "=" * 40 + "\n"

        return receipt

    def save_receipt(self, receipt_string):
        try:

            with open("transactions.txt", "a") as f:
                f.write(receipt_string)
                f.write("\n\n" + "*" * 50 + "\n\n") # Separator for clarity in the log
            print(f"\n Transaction saved successfully to transactions.txt (ID: {self.receipt_id}).")
        except IOError as e:
            print(f" Error: Could not save file. {e}")




def display_menu(menu_data):
    print("\n" + "=" * 40)
    print("      TIU COFFEE EXPRESS - MENU")
    print("=" * 40)
    print(f"{'Code':<5}{'Item':<25}{'Price':>8}")
    print("-" * 40)
    for code, details in menu_data.items():

        print(f"{code:<5}{details['item']:<25}${details['price']:>7.2f}")
    print("-" * 40 + "\n")

def get_valid_input(prompt, data_type=str, range_check=None):
    while True:
        try:
            user_input = input(prompt).strip()
            if data_type == int:
                value = int(user_input)
                if range_check is not None and value <= range_check:
                    raise ValueError("Quantity must be greater than zero.")
                return value
            elif data_type == str:
                return user_input.upper()
        except ValueError as e:
            print(f"Invalid input. {e if str(e) else 'Please enter a valid number or selection.'}")
        except Exception as e:
            print(f"An unexpected error occurred: {e}")

def take_order(order_object):
     while True:
        choice = get_valid_input("Enter item code (1-5) or 'D' to finish order: ", str).upper()

        if choice == 'D':
            break

        if choice in MENU:

            quantity = get_valid_input(f"Enter quantity for {MENU[choice]['item']}: ", int, 0)
            order_object.add_item(choice, quantity)
        else:
            print("Invalid code. Please enter a valid item code (1-5) or 'D'.")

def apply_discount_prompt(order_object):
      has_code = get_valid_input("Do you have a promo code? (Y/N): ", str)
      if has_code == 'Y':
        code = input("Enter your promo code: ").strip().upper()
        order_object.apply_discount(code)
      else:
        print("No discount applied.")
def main():
    print("=" * 50)
    print("WELCOME TO THE TIU COFFEE EXPRESS BILLING SYSTEM")
    print("=" * 50)
    customer_name = input("Enter Customer Name/ID: ").strip()
    current_order = CoffeeOrder(customer_name)
    display_menu(MENU)
    take_order(current_order)

    if not current_order.items:
        print("\n Order was empty. Exiting system.")
        return
    apply_discount_prompt(current_order)
    final_total = current_order.calculate_total()
    receipt_output = current_order.generate_receipt()
    print("\n" + "=" * 50)
    print("           ")
    print(receipt_output)
    current_order.save_receipt(receipt_output)
if __name__ == "__main__":
    main()