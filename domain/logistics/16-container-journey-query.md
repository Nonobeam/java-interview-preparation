## D16: How would you write a query to track a container's full journey (all port stops in order)?

JOIN container → booking → port_call → port, ordered by sequence_number, with ETA/ATA/ETD/ATD columns.
