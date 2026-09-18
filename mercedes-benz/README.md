# Mercedes-Benz Test Drive Booking — Exploratory Test Charter

Comprehensive exploratory testing session analyzing the Mercedes-Benz online Test Drive booking system's **input validation, error handling, and duplicate prevention** mechanisms.

---

## 📋 Overview

**Application:** [Mercedes-Benz Test Drive Booking](https://www.mercedes-benz.co.uk/passengercars/mercedes-benz-cars/online-testdrive.html)  
**Tester:** Sadgi Nayak  
**Date:** 10.09.2026  
**Duration:** 90 minutes  
**Approach:** Black-box exploratory testing with DevTools network inspection  

---

## 🎯 Mission

Determine whether the booking system's input validation, error handling, and duplicate prevention are enforced **server-side**, and whether failures are communicated to the user.

### Why This Matters

Test Drive booking is a lead-generation funnel. If validation is weak, errors are masked, and duplicate prevention doesn't exist server-side, customers and dealers will experience:
- Data confusion (invalid bookings accepted)
- Lost bookings (no duplicate detection)
- Poor UX (errors hidden from users)

---
