# CounselConnect: A Student Guidance Counseling Scheduling System

A C++ console program that emulates how a school Guidance Counseling Office schedules student appointments, built as the Final Project for **CPE 010 - Data Structures and Algorithms**.

![C++17](https://img.shields.io/badge/C%2B%2B-17-blue) ![Platform](https://img.shields.io/badge/app-console-lightgrey) ![Course](https://img.shields.io/badge/course-CPE%20010-green)

## Course Information

| Item | Details |
|---|---|
| Course | CPE 010 - Data Structures and Algorithms |
| Section / Group | CPE 010 - CPE21S4 |
| Instructor | Engr. Jimlord M. Quejado  |
| School / Department | Technological Institute of the Philippines - Quezon City (Computer Engineering Department |
| Date Submitted | November 6, 2026 |

## Team Members

| # | Full Name (Surname, First Name M.I.) | Student No. | Role | Main Contributions |
|---|---|---|---|---|
| 1 | Cardenas, Lea Marie N. | 2420071 | Assistant Leader | Project Idea, Brainstorming |
| 2 | De Vera, Sophia D. | 2410229 | Project Leader | Project Idea, Brainstorming |
| 3 | Hare, Andrei Angelo D. | 2012261 | QA | [Documentation, Github Repository Editor, Testing |
| 4 | Pangyarihan, Jenny Rose | 2420070 | Assistant Code Developer | Documentation, Objectives, Coder  |
| 5 | Peñaflorida, Wrio Carlo | 1512182 | Code Developer | Programming, Main Coding File Creator, Github Repository |

## Table of Contents
1. [Overview](#overview)
2. [Features](#features)
3. [Data Structures and Algorithms](#data-structures-and-algorithms)
4. [Performance](#performance)
5. [Getting Started](#getting-started)
6. [Sample Run](#sample-run)
7. [Project Structure](#project-structure)
8. [Source Code](#source-code)
9. [How to Upload Files](#how-to-upload-files-to-this-repository)
10. [Limitations](#limitations)
11. [Future Improvements](#future-improvements)
12. [AI Use Disclosure](#ai-use-disclosure)

## Overview

Students book guidance counseling sessions of 30 to 120 minutes (in 30-minute steps) between 8:30 AM and 5:30 PM. The system keeps each day's appointments sorted by start time, rejects overlapping bookings, and places students on a first-in, first-out waitlist when their requested time is taken. When an appointment is cancelled, waiting requests are automatically promoted.

**Overlap rule:** two appointments conflict when `NewStart < ExistingEnd AND NewEnd > ExistingStart`. Back-to-back sessions (for example 11:00-12:00 then 12:00-1:00) are allowed.

## Features

- Book an appointment with date, start time, and duration
- Validation of duration, 30-minute grid, and office hours
- Conflict detection against neighboring appointments
- Waitlist with automatic promotion on cancellation
- Cancel an appointment by ID
- View a day's schedule (student view shows only "Booked" to protect privacy)
- Invalid input handling (non-numeric input, bad menu choice, out-of-range times)

## Data Structures and Algorithms

| Topic | Where it is used |
|---|---|
| Dynamic array (`vector`) | Each day's appointments, kept sorted by start time |
| Binary search | Finds the insert position and the two neighbors to check for overlap |
| Linear search | Finds a day record or an appointment by ID |
| Queue | FIFO waitlist for requests that hit a conflict |
| Sorting (sorted insertion) | Keeps each day ordered as new bookings are added |

## Performance

| Operation | Technique | Time complexity |
|---|---|---|
| Book | Linear day lookup + binary search + sorted insert | O(d + log k + k) |
| Cancel | Linear search by ID, then erase | O(n) |
| View day | Linear day lookup + traversal | O(d + k) |
| Join waitlist | Queue enqueue | O(1) |

`n` = total appointments, `d` = number of days with bookings, `k` = appointments in one day (at most 18).

Experimental timings (n = 100 to 100,000) come from `src/benchmark.cpp`. See Section 5 of the project document for the full analysis and graph.

## Getting Started

**Requirements:** a C++17 compiler (g++ 9 or newer recommended).

```bash
# Clone
git clone https://github.com/[username]/CounselConnect-A-Student-Guidance-Counseling-Scheduling-System.git
cd CounselConnect-A-Student-Guidance-Counseling-Scheduling-System

# Build and run the program
g++ -std=c++17 -O2 src/main.cpp -o counselconnect
./counselconnect

# Build and run the benchmark
g++ -std=c++17 -O2 src/benchmark.cpp -o benchmark
./benchmark
```

On Windows, run `counselconnect.exe` and `benchmark.exe` instead.

## Sample Run

Input (book a 1-hour session, try an overlapping one, then view the day):

```
1
S2024-001
20261104
1030
60
1
S2024-002
20261104
1100
60
1
3
20261104
4
```

Expected result: the first booking succeeds, the second reports a conflict and offers the waitlist, and the day view lists `10:30 AM - 11:30 AM`.

Screenshots of each feature are in the [`docs/`](docs/) folder.

## Project Structure

```
CounselConnect-A-Student-Guidance-Counseling-Scheduling-System/
├── src/
│   ├── scheduler.h      # core scheduling logic
│   ├── main.cpp         # console menu
│   └── benchmark.cpp    # experimental timing code
├── docs/                # flowcharts and output screenshots
├── README.md
└── .gitignore
```

## Source Code

Click a file name to expand it. Each block has a copy button in its top-right corner.

<details>
<summary><b>src/main.cpp</b> (console menu)</summary>

```cpp
// main.cpp - console menu for CounselConnect: A Student Guidance Counseling Scheduling System
#include <limits>
#include <cstdlib>
#include "scheduler.h"

int readInt(const string& prompt) {
    int x;
    cout << prompt;
    while (!(cin >> x)) {
        if (cin.eof()) exit(0);
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Invalid input. " << prompt;
    }
    return x;
}

int main() {
    Scheduler s;
    cout << "==============================================\n"
         << "  CounselConnect\n"
         << "  A Student Guidance Counseling Scheduling System\n"
         << "  Office hours: 8:30 AM - 5:30 PM\n"
         << "==============================================\n";
    while (true) {
        cout << "\n1 Book  2 Cancel  3 View day  4 Exit\n";
        int c = readInt("Choice: ");
        if (c == 1) {
            Request r; cout << "Student ID: "; cin >> r.studentId;
            r.date = readInt("Date (YYYYMMDD): ");
            r.start = readInt("Start (24h HHMM, e.g. 1030): ");
            r.start = (r.start / 100) * 60 + r.start % 100;
            r.duration = readInt("Duration (minutes): ");
            string err; int id = s.book(r, err);
            if (id != -1) cout << "Booked. Appointment ID " << id << "\n";
            else {
                cout << "Failed: " << err << "\n";
                if (err.find("Conflicts") != string::npos && readInt("Join waitlist? (1/0): ") == 1) s.addToWaitlist(r);
            }
        } else if (c == 2) {
            cout << (s.cancel(readInt("Appointment ID: ")) ? "Cancelled.\n" : "ID not found.\n");
        } else if (c == 3) {
            s.showDay(readInt("Date (YYYYMMDD): "), false);
        } else if (c == 4) break;
        else cout << "Invalid choice.\n";
    }
}
```

</details>

<details>
<summary><b>src/scheduler.h</b> (core scheduling logic)</summary>

```cpp
// scheduler.h - CounselConnect core: sorted daily lists, binary search, linear search, waitlist queue
#ifndef SCHEDULER_H
#define SCHEDULER_H
#include <iostream>
#include <vector>
#include <queue>
#include <string>
using namespace std;

const int OPEN = 8 * 60 + 30, CLOSE = 17 * 60 + 30, STEP = 30, MAX_DUR = 120;

struct Appointment { int id; string studentId; int date; int start; int end; };
struct Request { string studentId; int date; int start; int duration; };
struct Day { int date; vector<Appointment> appts; };   // appts kept sorted by start time

inline string fmt(int m) {
    int h = m / 60, mm = m % 60, h12 = (h % 12 == 0) ? 12 : h % 12;
    return to_string(h12) + ":" + (mm < 10 ? "0" : "") + to_string(mm) + (h >= 12 ? " PM" : " AM");
}

class Scheduler {
    vector<Day> days;            // dynamic array of day records
    queue<Request> waitlist;     // FIFO for requests that hit a conflict
    int nextId = 1;

    // Linear search: find a day record by date. O(d)
    Day* findDay(int date) {
        for (Day& d : days) if (d.date == date) return &d;
        return nullptr;
    }

    // Binary search: first index whose start >= given start. O(log n)
    static int lowerBound(const vector<Appointment>& v, int start) {
        int lo = 0, hi = (int)v.size();
        while (lo < hi) {
            int mid = (lo + hi) / 2;
            if (v[mid].start < start) lo = mid + 1; else hi = mid;
        }
        return lo;
    }

public:
    // Returns appointment ID, or -1 with err set.
    int book(const Request& r, string& err) {
        int end = r.start + r.duration;
        if (r.duration <= 0 || r.duration > MAX_DUR || r.duration % STEP != 0) { err = "Duration must be 30-120 min in 30-min steps."; return -1; }
        if (r.start % STEP != 0) { err = "Start time must be on a 30-min boundary."; return -1; }
        if (r.start < OPEN || end > CLOSE) { err = "Session must fit within 8:30 AM - 5:30 PM."; return -1; }

        Day* day = findDay(r.date);
        if (!day) { days.push_back({r.date, {}}); day = &days.back(); }
        vector<Appointment>& v = day->appts;

        int pos = lowerBound(v, r.start);
        // Sorted + non-overlapping, so only the two neighbors need checking.
        // Overlap rule: NewStart < ExistingEnd AND NewEnd > ExistingStart
        if (pos > 0 && v[pos - 1].end > r.start) { err = "Conflicts with an earlier booking."; return -1; }
        if (pos < (int)v.size() && v[pos].start < end) { err = "Conflicts with a later booking."; return -1; }

        Appointment a{nextId++, r.studentId, r.date, r.start, end};
        v.insert(v.begin() + pos, a);            // keeps the day sorted. O(n) shift
        return a.id;
    }

    // Linear search by appointment ID across all days, then erase. O(n)
    bool cancel(int id) {
        for (Day& d : days) {
            for (size_t i = 0; i < d.appts.size(); i++) {
                if (d.appts[i].id == id) {
                    d.appts.erase(d.appts.begin() + i);
                    promoteWaitlist();
                    return true;
                }
            }
        }
        return false;
    }

    void addToWaitlist(const Request& r) { waitlist.push(r); }

    // One FIFO pass: try each waiting request; re-queue the ones that still don't fit.
    void promoteWaitlist() {
        int n = (int)waitlist.size();
        while (n--) {
            Request r = waitlist.front(); waitlist.pop();
            string err;
            int id = book(r, err);
            if (id != -1) cout << "[Waitlist] " << r.studentId << " booked, ID " << id << "\n";
            else waitlist.push(r);
        }
    }

    void showDay(int date, bool privateView) {
        Day* d = findDay(date);
        if (!d || d->appts.empty()) { cout << "No bookings.\n"; return; }
        for (const Appointment& a : d->appts) {
            cout << fmt(a.start) << " - " << fmt(a.end) << "  ";
            if (privateView) cout << "Booked\n";      // students never see other students' details
            else cout << a.studentId << " (ID " << a.id << ")\n";
        }
    }
};

#endif
```

</details>

<details>
<summary><b>src/benchmark.cpp</b> (experimental timing)</summary>

```cpp
// benchmark.cpp - experimental timing for Section 5 (book and cancel vs. input size n)
#include <chrono>
#include "scheduler.h"
using namespace std::chrono;

int main() {
    const int sizes[] = {100, 1000, 10000, 100000};
    const int REPS = 5, OPS = 100;
    cout << "n,book_ms,cancel_ms\n";
    for (int n : sizes) {
        double bookTotal = 0, cancelTotal = 0;
        for (int rep = 0; rep < REPS; rep++) {
            Scheduler s;
            string err;
            // Preload n
