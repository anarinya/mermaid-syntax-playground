```mermaid
classDiagram
  Brand -- Hotel

  Hotel *-- Room
  Room *-- RoomType

  Guest *-- Booking
  Room *-- Booking
  Hotel *-- Booking
  RoomType *-- RateType

  Hotel o-- Guest
  Hotel o-- Employee
  Booking o-- RateType
```