# Hotel Booking System - UML Class Diagram

UML Class Diagram Modelling Assignment | Kigali Independent University (ULK) | Instructor: Eric Maniraguha

**Group members:** Maysara Dafaallh (202512952) and Ahmed Alhadi (202513597)

## Business idea
A hotel needs an information system to manage its clients, rooms, bookings and payments, replacing paper records and spreadsheets that cause double bookings, lost records and weak payment tracking.

## Scope
**Covered:** client registration and profile update; room details and availability; creating and cancelling bookings; processing payments and generating receipts.

**Excluded:** staff/HR management, housekeeping and inventory, restaurant and other services, online payment gateway integration, advanced reporting, and software implementation (this project is a design only).

## Classes
| Class | Attributes | Methods |
|---|---|---|
| **Client** | clientId: int, fullName: String, email: String, phone: String | register(), updateProfile() |
| **Room** | roomId: int, roomType: String, pricePerNight: double, isAvailable: boolean | checkAvailability(), updateStatus() |
| **Booking** | bookingId: int, checkInDate: String, checkOutDate: String, status: String | createBooking(), cancelBooking() |
| **Payment** | paymentId: int, amount: double, paymentDate: String, paymentMethod: String | processPayment(), generateReceipt() |

## Relationships and multiplicities
| Association | Name | End 1 | End 2 |
|---|---|---|---|
| Client – Booking | makes | Client: 1 | Booking: 0..* |
| Booking – Room | reserves | Booking: 0..* | Room: 1 |
| Booking – Payment | is settled by | Booking: 1 | Payment: 0..1 |

All relationships are simple associations (no inheritance). Assumptions: one room per booking, one payment per booking, no overlapping bookings of the same room.

## Diagram
![UML Class Diagram](diagrams/uml_class_diagram.png)

## Repository structure
```
uml-class-diagram-assignment/
|-- README.md
|-- documentation/
|   |-- report_1_project_description.pdf
|   `-- report_2_uml_class_diagram.pdf
`-- diagrams/
    |-- uml_class_diagram.drawio
    `-- uml_class_diagram.png
```
