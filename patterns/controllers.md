# Controllers

## Overview

Controllers should be thin and contain very little logic.  Controllers should generally have the following functionality:

1. Read in the params
2. Persist Data
3. Return result (or error)

Controllers, workers, and form objects are considered safe places to persist data to the database.  Persisting data in the controller allows us to return a consistent result to the the client without having "surprise side effects" elsewhere in the code.

Each controller should generally only use a single transaction. If multiple records must be updated, a transaction block can be used to ensure that all of the changes are atomic.

## Testing

Controllers are tested through request specs.  We no longer implement controller specs. &#x20;

## Example
