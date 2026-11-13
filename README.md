# hotel-management-system

#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <limits>
using namespace std;

// Room Class
class Room {
public:
    int roomNo;
    string type;
    double price;

    Room(int no, string t, double p) {
        roomNo = no;
        type = t;
        price = p;
    }
};

// Customer Class
class Customer {
public:
    int id;
    string name;
    string phone;

    Customer(int i, string n, string p) {
        id = i;
        name = n;
        phone = p;
    }
};

// Booking Class
class Booking {
public:
    int bookingId;
    int roomNo;
    int customerId;
    string name;
    string phone;
    bool active;
};

// Hotel Management Class
class HotelManagement {
private:
    vector<Room> rooms;
    vector<Booking> bookings;
    int nextBookingId = 1;

    // Save records to file
    void saveData() {
        ofstream file("hotel.txt");

        for (auto b : bookings) {
            file << b.bookingId << "|"
                 << b.roomNo << "|"
                 << b.customerId << "|"
                 << b.name << "|"
                 << b.phone << "|"
                 << b.active << endl;
        }

        file.close();
    }

    // Load records from file
    void loadData() {
        ifstream file("hotel.txt");

        if (!file)
            return;

        Booking b;
        string line;

        while (getline(file, line)) {
            size_t pos = 0;
            string fields[6];
            int count = 0;

            while ((pos = line.find('|')) != string::npos) {
                fields[count++] = line.substr(0, pos);
                line.erase(0, pos + 1);
            }

            fields[count++] = line;

            if (count == 6) {
                b.bookingId = stoi(fields[0]);
                b.roomNo = stoi(fields[1]);
                b.customerId = stoi(fields[2]);
                b.name = fields[3];
                b.phone = fields[4];
                b.active = stoi(fields[5]);

                bookings.push_back(b);

                if (b.bookingId >= nextBookingId)
                    nextBookingId = b.bookingId + 1;
            }
        }

        file.close();
    }

    bool isRoomAvailable(int roomNo) {
        for (auto b : bookings) {
            if (b.roomNo == roomNo && b.active)
                return false;
        }
        return true;
    }

public:
    HotelManagement() {
        rooms.push_back(Room(101, "Single", 1000));
        rooms.push_back(Room(102, "Double", 1500));
        rooms.push_back(Room(103, "Deluxe", 2500));
        rooms.push_back(Room(104, "Single", 1000));
        rooms.push_back(Room(105, "Deluxe", 2500));

        loadData();
    }

    // Display Rooms
    void displayRooms() {
        cout << "\n------ ROOM DETAILS ------\n";

        for (auto r : rooms) {
            cout << "Room Number: " << r.roomNo
                 << "\nType: " << r.type
                 << "\nPrice: Rs." << r.price
                 << "\nStatus: "
                 << (isRoomAvailable(r.roomNo)
                     ? "Available" : "Booked")
                 << "\n----------------------\n";
        }
    }

    // Book Room
    void bookRoom() {
        int roomNo, id;
        string name, phone;

        displayRooms();

        cout << "\nEnter Room Number: ";
        cin >> roomNo;

        if (!isRoomAvailable(roomNo)) {
            cout << "Sorry! Room already booked.\n";
            return;
        }

        bool found = false;

        for (auto r : rooms) {
            if (r.roomNo == roomNo)
                found = true;
        }

        if (!found) {
            cout << "Invalid Room Number!\n";
            return;
        }

        cout << "Enter Customer ID: ";
        cin >> id;
        cin.ignore(numeric_limits<streamsize>::max(), '\n');

        cout << "Enter Customer Name: ";
        getline(cin, name);

        cout << "Enter Phone Number: ";
        getline(cin, phone);

        Booking b;
        b.bookingId = nextBookingId++;
        b.roomNo = roomNo;
        b.customerId = id;
        b.name = name;
        b.phone = phone;
        b.active = true;

        bookings.push_back(b);

        saveData();

        cout << "\nRoom Booked Successfully!\n";
        cout << "Booking ID: " << b.bookingId << endl;
    }

    // Checkout
    void checkout() {
        int roomNo;

        cout << "\nEnter Room Number for Checkout: ";
        cin >> roomNo;

        for (auto &b : bookings) {
            if (b.roomNo == roomNo && b.active) {
                b.active = false;
                saveData();

                cout << "\nCheckout Successful!\n";
                cout << "Customer Name: " << b.name << endl;
                cout << "Room Number: " << b.roomNo << endl;
                return;
            }
        }

        cout << "No Active Booking Found!\n";
    }

    // Search Customer
    void searchCustomer() {
        int id;

        cout << "\nEnter Customer ID: ";
        cin >> id;

        for (auto b : bookings) {
            if (b.customerId == id) {
                cout << "\nCustomer Details\n";
                cout << "Booking ID: " << b.bookingId << endl;
                cout << "Customer ID: " << b.customerId << endl;
                cout << "Name: " << b.name << endl;
                cout << "Phone: " << b.phone << endl;
                cout << "Room Number: " << b.roomNo << endl;
                cout << "Status: "
                     << (b.active ? "Active" : "Checked Out")
                     << endl;
            }
        }
    }

    // Display All Bookings
    void displayBookings() {
        cout << "\n------ BOOKING DETAILS ------\n";

        if (bookings.empty()) {
            cout << "No bookings available.\n";
            return;
        }

        for (auto b : bookings) {
            cout << "\nBooking ID: " << b.bookingId;
            cout << "\nCustomer Name: " << b.name;
            cout << "\nRoom Number: " << b.roomNo;
            cout << "\nPhone: " << b.phone;
            cout << "\nStatus: "
                 << (b.active ? "Active" : "Checked Out");
            cout << "\n------------------------\n";
        }
    }

    // Main Menu
    void menu() {
        int choice;

        do {
            cout << "\n===== HOTEL MANAGEMENT SYSTEM =====\n";
            cout << "1. Display Rooms\n";
            cout << "2. Book Room\n";
            cout << "3. Checkout\n";
            cout << "4. Search Customer\n";
            cout << "5. Display All Bookings\n";
            cout << "6. Exit\n";

            cout << "\nEnter Your Choice: ";
            cin >> choice;

            switch (choice) {
                case 1:
                    displayRooms();
                    break;

                case 2:
                    bookRoom();
                    break;

                case 3:
                    checkout();
                    break;

                case 4:
                    searchCustomer();
                    break;

                case 5:
                    displayBookings();
                    break;

                case 6:
                    cout << "\nThank You!\n";
                    break;

                default:
                    cout << "Invalid Choice! Try Again.\n";
            }

        } while (choice != 6);
    }
};

// Main Function
int main() {
    HotelManagement hotel;
    hotel.menu();

    return 0;
}
