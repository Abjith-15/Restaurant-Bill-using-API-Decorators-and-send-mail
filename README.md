# Restaurant-Bill-using-API-Decorators-and-send-mail
import requests
import day_30
import datetime

# 🔹 Decorator for authentication
def login_required(func):
    def wrapper(*args, **kwargs):
        password = input("Enter admin password: ")
        if password == "12345":
            return func(*args, **kwargs)
        else:
            print("❌ Wrong password")
    return wrapper


class HotelBilling:

    def __init__(self, data):
        self.name = data.get('name')
        self.email = data.get('email')
        self.date = datetime.datetime.now()

        self.menu = {
            "biryani": 120,
            "fried_rice": 100,
            "noodles": 90,
            "chicken65": 110
        }

    def show_menu(self):
        print("\n📋 MENU CARD")
        for item, price in self.menu.items():
            print(f"{item} - ₹{price}")

    def take_order(self):
        total_bill = 0
        orders = []

        while True:
            order = input("\nEnter item (or type 'done'): ").lower()

            if order == "done":
                break

            if order not in self.menu:
                print("❌ Item not available")
                continue

            try:
                qty = int(input("Enter quantity: "))
            except:
                print("❌ Invalid quantity")
                continue

            price = self.menu[order]
            total = price * qty
            total_bill += total

            orders.append((order, qty, total))

            print(f"✅ Added {order} x{qty} = ₹{total}")

        return orders, total_bill

    @login_required
    def generate_bill(self, orders, total):
        print("\n🧾 GENERATING BILL...")

        bill_text = "------ HOTEL BILL ------\n"
        bill_text += f"Name: {self.name}\n"
        bill_text += f"Date: {self.date}\n\n"

        for item, qty, price in orders:
            bill_text += f"{item} x{qty} = ₹{price}\n"

        bill_text += f"\nTotal Bill: ₹{total}\n"

        print(bill_text)

        choice = input("Send mail? (yes/no): ").lower()

        if choice == "yes":
            day_30.mail(self.email, self.name, "Multiple Items", total)
            print("📧 Mail sent")
        else:
            with open("FINAL_BILL.txt", "w") as f:
                f.write(bill_text)
            print("💾 Bill saved to file")


def main():
    url = "http://demo4728352.mockable.io/Abjith.mini.py.lead"

    try:
        res = requests.get(url)

        if res.status_code == 200:
            data = res.json()

            hotel = HotelBilling(data)

            hotel.show_menu()

            orders, total = hotel.take_order()

            if total > 0:
                hotel.generate_bill(orders, total)
            else:
                print("❌ No orders placed")

        else:
            print("❌ API Error")

    except Exception as e:
        print("❌ Something went wrong:", e)


main()
