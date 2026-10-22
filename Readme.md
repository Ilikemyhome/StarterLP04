# Learning Practice: Black-Box Testing, Part 2


## Step 1: Identify Conditions and Actions

### Condition Examples

- The check-in and checkout dates are valid.
- The reservation is between 1 and 31 nights.
- The reservation includes between 1 and 12 guests.
- Every requested night is available.
- The reservation includes more than five guests.

### Action Examples

- Create the reservation.
- Calculate the standard reservation price.
- Add the extra guest fee.
- Raise a `ValueError`.
- Raise an `UnavailableError`.

## Step 2: Create a Decision Table

Your table should follow this structure:

| Conditions and Actions         | Rule 1 | Rule 2 | Rule 3 | Rule 4 | Rule 5 | Rule 6 |
| ------------------------------ | :----: | :----: | :----: | :----: | :----: | :----: |
| Dates are valid                |   T    |   T    |   F    |   T    |   T    |   T    |
| Number of nights is valid      |   T    |   T    |   -    |   F    |   T    |   T    |
| Number of guests is valid      |   T    |   T    |   -    |   -    |   F    |   T    |
| Requested nights are available |   T    |   T    |   -    |   -    |   -    |   F    |
| More than five guests          |   F    |   T    |   -    |   -    |   -    |   -    |
| **Create reservation**         |   X    |   X    |        |        |        |        |
| **Calculate standard price**   |   X    |   X    |        |        |        |        |
| **Add extra guest fee**        |        |   X    |        |        |        |        |
| **Raise `ValueError`**         |        |        |   X    |   X    |   X    |        |
| **Raise `UnavailableError`**   |        |        |        |        |        |   X    |

Rule 1: Valid reservation, 5 or fewer guests.
Rule 2: Valid reservation, more than 5 guests.
Rule 3: Invalid dates.
Rule 4: Invalid number of nights.
Rule 5: Invalid number of guests.
Rule 6: Dates unavailable.

## Step 3: Create Decision Table Test Cases

### TC01: Valid Reservation With No Extra Guest Fee
- **Check-in:** October 1
- **Checkout:** October 4
- **Guests:** 4
- **Availability:** Available.
- **Expected Result:** Reservation  created.
- **Expected Price:**
    - 3 * $200 = $600
    - Cleaning fee = $50
    - Total = $650

### TC02: Valid Reservation With Extra Guest Fee
- **Check-in:** October 1
- **Checkout:** October 4
- **Guests:** 7
- **Availability:** Available
- **Expected Action:** Reservation created with extra guest fee
- **Expected Price:**
    - 3 × $200 = $600
    - Cleaning fee = $50
    - Extra guests: (7 − 5) × $25 × 3 = $150
    - Total = $800

### TC03: Invalid Dates
- **Check-in:** October 10
- **Checkout:** October 8
- **Guests:** 4
- **Availability:** Available
- **Expected Action:** Raise ValueError
- **Expected Exception:** ValueError

### TC04: Invalid Number of Nights
- **Check-in:** October 1
- **Checkout:** November 5
- **Guests:** 4
- **Availability:** Available
- **Expected Action:** Raise ValueError
- **Expected Exception:** ValueError

### TC05: Invalid Number of Guests
- **Check-in:** October 1
- **Checkout:** October 4
- **Guests:** 13
- **Availability:** Available
- **Expected Action:** Raise ValueError
- **Expected Exception:** ValueError

### TC06: Unavailable Reservation
- **Check-in:** October 1
- **Checkout:** October 4
- **Guests:** 4
- **Availability:** One night unavailable
- **Expected Action:** Raise UnavailableError
- **Expected Exception:** UnavailableError

## Step 4: Identify Reservation States and Events

### Reservation States

- `Pending`
- `Confirmed`
- `Checked In`
- `Completed`
- `Cancelled`

### Events That Change Reservation States

- Payment succeeds
- Payment fails
- Guest cancels
- Guest checks in
- Guest checks out

## Step 5: Create a State Transition Table

|   Current State   |	Event           |	Expected State  |	Expected Result          |
|-------------------|-------------------|-------------------|----------------------------|
|   Pending         | Payment succeeds	| Confirmed	        | Reservation is confirmed   |
|   Pending     	| Payment fails 	| Cancelled	        | Reservation is cancelled   |
|   Pending	        | Guest cancels 	| Cancelled	        | Reservation is cancelled   |
|   Confirmed       | Guest checks in	| Checked In	    | Guest is checked in        |
|   Confirmed	    | Guest cancels	    | Cancelled	        | Reservation is cancelled   |
|   Checked In  	| Guest checks out	| Completed	        | Reservation is completed   |
|   Cancelled	    | Payment succeeds	| Cancelled	        | InvalidStateError          |
|   Completed       | Guest cancels	    | Completed	        | InvalidStateError          |
|   Checked In      | Payment succeeds  | Checked In        | InvalidStateError          |

## Step 6: Create State Transition Test Cases

|  Test ID |   Starting State   	|   Event	     |Expected State       |   Expected Result   |	Type    |
|----------|------------------------|----------------|---------------------|---------------------|----------|
| ST01	   |Pending	                |Payment succeeds|	Confirmed	       |Reservation confirmed|	Valid   |
| ST02	   |Pending	                | Payment fails	 |  Cancelled	       |Reservation cancelled|	Valid   |
| ST03	   |Pending	                | Guest cancels	 | Cancelled	       |Reservation cancelled|	Valid   |
| ST04	   |Confirmed               | Guest checks in|	Checked In	       |Guest checked in	 |  Valid   |
| ST05	   |Confirmed	            | Guest cancels  | Cancelled	       |Reservation cancelled|  Valid   |
| ST06	   |Checked In	            |Guest checks out|	Completed	       |Reservation completed|	Valid   |
| ST07	   |Cancelled	            |Payment succeeds|	Cancelled	       |InvalidStateError	 | Invalid  |
| ST08     |Completed	            | Guest cancels  |	Completed	       |InvalidStateError	 | Invalid  |
| ST09     |Checked In              |Payment succeeds|	Checked In         |InvalidStateError	 | Invalid  |
| ST10     |Pending -> Confirmed -> Checked In -> Completed  |	Full reservation sequence |	Completed	| Reservation completed successfully |	Valid Sequence |
| ST11	   |Pending -> Confirmed → Cancelled	| Cancellation sequence|	Cancelled	Reservation|  cancelled successfully |	Valid Sequence|
