<div align="center">

# 🎬 Cinema Management System

**A console-based cinema seat management system built in Java**

![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![OOP](https://img.shields.io/badge/OOP-Principles-blue?style=for-the-badge)
![Package](https://img.shields.io/badge/Package-sp25__bcs__133-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

*Manage cinemas, screens, and seats across multiple cities — all from the console.*

</div>

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Class Reference](#-class-reference)
  - [Type](#-type--enum)
  - [Seat](#-seat)
  - [Screen](#-screen)
  - [Cinema](#-cinema)
  - [CityCinema](#-citycinema)
  - [CinemaDemo](#-cinemademo)
- [Seat Layout](#-seat-layout)
- [Seat ID Format](#-seat-id-format)
- [Known Issues](#-known-issues)

---

## 🗺️ Overview

The Cinema Management System is a Java project that models a real-world cinema chain. It supports:

- Multiple **cities**, each containing multiple **cinemas**
- Each cinema contains multiple **screens**, each with a unique **seat layout**
- Seats are categorized by **type** (Regular, Premium, VIP, Recliner) with different prices
- Seats can be **booked** and **cancelled** individually by ID
- The system can search for the **first available seat** of a given type across an entire city

---

## 🏗️ Architecture

```
CityCinema          — a city containing multiple cinemas
    └── Cinema[]    — a cinema containing multiple screens
            └── Screen[]    — a screen containing a jagged 2D seat grid
                    └── Seat[][]    — individual seats with ID, type, price, availability
```

```
Type (enum)
    REGULAR | PREMIUM | VIP | RECLINER
```

### Class Dependency Diagram

```
CinemaDemo
    └── CityCinema
            └── Cinema
                    └── Screen
                            └── Seat
                                  └── Type (enum)
```

---

## 📚 Class Reference

---

### 🎫 `Type` — Enum

> `Type.java`

Defines the four seat categories available in every screen. Used throughout `Screen` and `Seat` for type-based filtering and pricing.

```java
public enum Type {
    REGULAR, PREMIUM, VIP, RECLINER;
}
```

| Value | Price (PKR) | Position in Screen |
|-------|------------|---------------------|
| `REGULAR` | 500 | First third of rows |
| `PREMIUM` | 750 | Middle third of rows |
| `VIP` | 1000 | Second-to-last row |
| `RECLINER` | 1200 | Last row |

---

### 💺 `Seat`

> `Seat.java`

Represents a single physical seat. Stores the seat's identity, type, price, and booking status. Tracks availability with both a `boolean` and a `String` label (`"A"` / `"NA"`).

#### Constructors

```java
Seat(String id, Type type, double price, boolean isAvailable)
Seat()   // empty constructor
```

| Parameter | Description |
|-----------|-------------|
| `id` | Unique seat identifier in `row-col` format (e.g. `"2-003"`) |
| `type` | One of `REGULAR`, `PREMIUM`, `VIP`, `RECLINER` |
| `price` | Ticket price in PKR |
| `isAvailable` | `true` if the seat can be booked |

> **Note:** Availability is stored in two separate fields — `isAvailable` (boolean) and `availability` (String). Both must be updated together using `setBoolAvailability()` and `setAvailability()`.

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | `String` | Unique seat ID |
| `type` | `Type` | Seat category |
| `price` | `double` | Ticket price |
| `isAvailable` | `boolean` | Booking status |
| `availability` | `String` | `"A"` = available, `"NA"` = not available |

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getID()` | `String` | Returns the seat ID |
| `getPrice()` | `double` | Returns the ticket price |
| `getAvailability()` | `boolean` | Returns `true` if the seat is available |
| `getType()` | `Type` | Returns the seat type |
| `setBoolAvailability(boolean)` | `void` | Sets the boolean availability flag |
| `setAvailability(String)` | `void` | Sets the String availability label (`"A"` or `"NA"`) |
| `displaySeating()` | `String` | Returns `[id:A]` or `[id:NA]` — used for grid display |
| `toString()` | `String` | Returns `[id:type:price:availability]` |

---

### 🖥️ `Screen`

> `Screen.java`

Represents a single cinema screen. Contains a **jagged 2D array** of `Seat` objects — each row has more seats than the one before it, modelling the stepped layout of a real cinema.

#### Constructor

```java
public Screen(String screenName, int totalRows)
```

| Parameter | Description |
|-----------|-------------|
| `screenName` | Display name, e.g. `"SCREEN-1"` |
| `totalRows` | Number of seat rows (each row `i` has `i + 5` seats) |

#### Seat Layout Logic

Each row `i` gets `i + 5` seats. The type and price assigned to each row depends on its position relative to `totalRows`:

```
Row index < totalRows / 3          → REGULAR   (PKR 500)
Row index < (2 * totalRows) / 3    → PREMIUM   (PKR 750)
Row index == totalRows - 2         → VIP       (PKR 1000)
Everything else (last row)         → RECLINER  (PKR 1200)
```

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `seats` | `Seat[][]` | Jagged 2D array of all seats in this screen |
| `screenName` | `String` | Display name of the screen |

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `bookSeat(String id)` | `Seat` | Books the seat with the given ID. Returns the booked `Seat`, or `null` if already booked or not found. Prints status to console. |
| `cancelBooking(String id)` | `void` | Cancels the booking for the given seat ID. Prints status to console. |
| `getTotalSeats()` | `int` | Returns total number of seats across all rows |
| `getAvailableSeats()` | `int` | Returns number of currently available seats |
| `getTotalSeatsByType(Type)` | `int` | Returns total seats of the given type |
| `getAvailableSeatsByType(Type)` | `int` | Returns available seats of the given type |
| `findFirstAvailableSeatByType(Type)` | `Seat` | Returns the first available seat of the given type, or `null` if none |
| `showScreen()` | `void` | Prints the full screen display: grid, totals, type breakdown, and seat details table |
| `displaySeats()` | `void` | Prints the visual seat grid using `[id:A]` / `[id:NA]` notation |
| `displaySeatDetails()` | `void` | Prints a formatted table of all seats with ID, type, price, and availability |
| `getName()` | `String` | Returns the screen name |

#### `bookSeat()` Behaviour

```
ID found + seat available   → books it, prints confirmation, returns Seat
ID found + already booked   → prints error, returns null
ID not found                → prints "not found", returns null
```

---

### 🏢 `Cinema`

> `Cinema.java`

Represents a single cinema building. Contains multiple `Screen` objects and maps each screen to a movie from a fixed list.

#### Constructor

```java
public Cinema(String name, int totalScreens)
```

Creates `totalScreens` screens named `SCREEN-1`, `SCREEN-2`, etc. Each screen is assigned a movie from `moviesPlaying[]` using modulo cycling — so if there are more screens than movies, the list repeats.

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | `String` | Cinema name (e.g. `"CINEMA-1"`) |
| `totalScreens` | `int` | Number of screens |
| `moviesPlaying` | `String[]` | Hardcoded pool of 5 movies |
| `screens` | `Screen[]` | Array of screen objects |
| `moviesForScreens` | `String[]` | Movie assigned to each screen |

#### Movie Pool

```java
{"Lion King", "Finding Nemo", "Kung-Fu Panda", "Spirited Away", "Harry Potter"}
```

Movies are assigned to screens by index: `screen[i]` plays `moviesPlaying[i % 5]`.

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `welcomeScreen()` | `void` | Prints all screens with their current movie and full seat layout |
| `findScreen(String name)` | `void` | Searches for a screen by name and prints its details |
| `getName()` | `String` | Returns the cinema name |
| `getMoviePlaying(int screenIndex)` | `String` | Returns the movie assigned to the screen at `screenIndex`. Returns `"Invalid screen index"` if out of range. |
| `getTotalCinemaSeats()` | `int` | Returns total seats across all screens in this cinema |
| `getAvailableCinemaSeats()` | `int` | Returns available seats across all screens |
| `getCinemaSeatsByType()` | `void` | Prints a breakdown of seat counts by type for every screen |
| `getScreen(int index)` | `Screen` | Returns the screen at `index`, or `null` if out of bounds |
| `getScreens()` | `Screen[]` | Returns the full array of screens |
| `toString()` | `String` | Returns `[name totalScreens totalSeats availableSeats]` |

---

### 🏙️ `CityCinema`

> `CityCinema.java`

The top-level container. Represents a city holding an array of `Cinema` objects. Provides city-wide operations like finding VIP seats, adding/removing cinemas, and displaying all cinema details.

#### Constructor

```java
CityCinema(String cityName, int cinemaNumber)
```

Creates `cinemaNumber` cinemas, each with 5 screens, named `CINEMA-1`, `CINEMA-2`, etc.

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `cinemas` | `Cinema[]` | Array of cinemas in this city |
| `cityName` | `String` | Name of the city (e.g. `"LAHORE"`) |

> **Note:** `cinemaNumber` was previously stored as a field but is now commented out. The array length serves as the source of truth for cinema count.

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `showAllCinemas()` | `void` | Prints a full city report — all cinemas and all their screen details |
| `findFirstAvailableVIPSeat()` | `void` | Searches all cinemas and screens in the city for the first available VIP seat. Prints seat ID, screen, and price. |
| `addCinema(String name, int totalScreens)` | `void` | Dynamically grows the `cinemas` array by 1 and adds the new cinema |
| `removeCinema(String name)` | `void` | Searches for a cinema by name (case-insensitive) and removes it by rebuilding the array without it |
| `findCinema(String name)` | `Cinema` | Returns the `Cinema` object matching the name, or `null` if not found |
| `getCinema(int index)` | `Cinema` | Returns the cinema at `index`, or `null` if out of bounds |

#### `addCinema()` — How it works

Since Java arrays are fixed-size, adding a cinema creates a new array of size `n+1`, copies all existing cinemas over, and appends the new one:

```
old array [A, B, C]
              ↓
new array [A, B, C, NEW]
```

#### `removeCinema()` — How it works

Finds the cinema index, creates a new array of size `n-1`, and copies all elements except the one to remove:

```
old array [A, B, C, D]   (removing B at index 1)
              ↓
new array [A, C, D]
```

---

### ▶️ `CinemaDemo`

> `CinemaDemo.java`

The entry point and demonstration class. Shows basic system usage.

```java
public static void main(String args[]) {
    CityCinema lahoreCinemas = new CityCinema("LAHORE", 1);
    lahoreCinemas.showAllCinemas();                              // print full report

    lahoreCinemas.getCinema(0).getScreen(0).bookSeat("1-001");  // book a seat
    lahoreCinemas.showAllCinemas();                              // print updated report
}
```

This creates one city (Lahore) with one cinema and five screens, shows the initial state, books seat `1-001` in the first screen, then prints the updated state to confirm the booking.

---

## 🪑 Seat Layout

Every screen uses a **jagged 2D array** — each row has more seats than the one above it. For a screen with `totalRows = 5`:

```
Row 1:  [1-001] [1-002] [1-003] [1-004] [1-005]              ← 5 seats  | REGULAR
Row 2:  [2-001] [2-002] [2-003] [2-004] [2-005] [2-006]      ← 6 seats  | REGULAR
Row 3:  [3-001] ... [3-007]                                   ← 7 seats  | PREMIUM
Row 4:  [4-001] ... [4-008]                                   ← 8 seats  | VIP
Row 5:  [5-001] ... [5-009]                                   ← 9 seats  | RECLINER
```

Row `i` always has `i + 5` seats, starting from `i = 0` (displayed as Row 1).

### Type Assignment (5 rows)

| Rows | Type | Price |
|------|------|-------|
| Row 1 – 1 (`i < 5/3 = 1`) | REGULAR | PKR 500 |
| Row 2 – 3 (`i < 10/3 = 3`) | PREMIUM | PKR 750 |
| Row 4 (`i == totalRows - 2`) | VIP | PKR 1000 |
| Row 5 (last row) | RECLINER | PKR 1200 |

---

## 🔖 Seat ID Format

Every seat has a unique ID in the format:

```
row-col
  │   └── 3-digit zero-padded column number
  └────── row number (1-indexed)
```

**Examples:**

| ID | Meaning |
|----|---------|
| `1-001` | Row 1, Column 1 |
| `3-007` | Row 3, Column 7 |
| `5-009` | Row 5, Column 9 |

IDs are generated in `Screen`'s constructor:
```java
String id = (i + 1) + "-" + String.format("%03d", j + 1);
```

---

## ⚠️ Known Issues

| # | Issue | Location | Detail |
|---|-------|----------|--------|
| 1 | Duplicate availability tracking | `Seat.java` | Availability is stored as both a `boolean` (`isAvailable`) and a `String` (`"A"/"NA"`). Both must be updated together manually — they can go out of sync if only one setter is called. |
| 2 | `moviesPlaying[]` is package-private | `Cinema.java` | The field has no access modifier, making it accessible to all classes in the package. It should be `private`. |
| 3 | `seats[][]` was previously package-private | `Screen.java` | Now correctly `private`, but `CinemaApp.java` (the JavaFX UI) accesses it directly as `screen.seats`. This breaks encapsulation. |
| 4 | Hardcoded movie list | `Cinema.java` | `moviesPlaying[]` is defined directly in the class with 5 fixed titles. Movies cannot be added or changed without editing source code. |
| 5 | Hardcoded screen count | `CityCinema.java` | Each cinema is always created with exactly 5 screens regardless of the `totalScreens` parameter — the constructor passes 5 directly to `new Cinema(...)`. |
| 6 | `cinemaNumber` field commented out | `CityCinema.java` | The constructor parameter is used to size the array but the field itself is commented out, leaving no way to query total cinema count later. |
| 7 | `findScreen()` silently does nothing | `Cinema.java` | If no screen matches the given name, the method returns without printing any message to the user. |

---

<div align="center">

Made with ☕ Java &nbsp;·&nbsp; Tayyaba Riaz &nbsp;·&nbsp;

</div>
