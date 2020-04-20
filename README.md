Cf. [https://www.plctalk.net/qanda/showthread.php?t=124552](https://www.plctalk.net/qanda/showthread.php?t=124552)

    Process
    =======
    Boxes on a conveyor
    Three independent discrete inputs:
    - Two PhotoEye (PE) station discrete inputs:  upstream BOX_AT_PE1; downstream BOX_AT_PE2
      - 1 => box is present at station; 0 => no box is present at station
    - A third discrete input REJECT_BOX_AT_PE1 determines whether a box at the upstream station either
      - is a reject (1), and to be later diverted at PE2,
      OR
      - is okay (0), and not to be diverted.
    Rising edge at upstream station BOX_AT_PE1 station is where each box's {1:reject;0:okay} status is set
    - Based on the status of REJECT_BOX_AT_PE1
    When each box later generates a rising edge at downstream station BOX_AT_PE2
    - divert if box status from BOX_AT_PE1 was [reject]
    - do not divert if box status BOX_AT_PE1 was [okay]
    There will be an arbitrary number of boxes that have triggered BOX_AT_PE1 but not yet triggered BOX_AT_PE2
    - These are the only boxes this program can keep track of with the available inputs
    - The number of boxes will range from 0 to no more than approximately 10

    Implementation data structures
    =========================
    Integer index TRACKED_BOX_COUNT (N199:0) is the count of tracked boxes
    - I.e boxes that have triggered BOX_AT_PE1 but not yet triggered BOX_AT_PE2
    Bit array (FIFO) in file #N99
    - Only up bits up to to N99:0/[TRACKED_BOX_COUNT-1] represent tracked boxes
    - N99 bit count = 64, whcih is much greater than the maximum possible value of TRACKED_BOX_COUNT
    - Bit value is 1 if a tracked box is a [reject] and is to be diverted at PE2
    - Bit value is 0 if a tracked box is [okay] and is not to be diverted at PE2
    - FIFO content and shifting are controlled with BSR only, not FFL/FFU
    - BSR always pushes 0-valued bit (status [okay]) at front of FIFO
      - Which will aways be well upstream of N99:0/[TRACKED_BOX_COUNT]

    Implementation events
    ==================
    At BOX_AT_PE2 rising edge
    - Pop reject/okay bit
      - Set (or leave) DIVERTED bit state to (as) that popped bit
    - Decrement TRACKED_BOX_COUNT
    At PE1 rising edge
    - Ensure bit N99:0/[TRACKED_BOX_COUNT] has same current value as REJECT_BOX_AT_PE1
    - Increment TRACKED_BOX_COUNT
     
    Implementation assumptions
    =======================
    There is adequate physical space between boxes, so there is exactly 1 rising edge per box at photoeyes, both at PE1, and at PE2
    No boxes are added or subtracted between PE1 and PE2, nor coffee cups blocking either PE

    Initialization
    =========
    Set diverted state to 0
    - Better alternative would be detect diverter state
    Set FIFO and index to zero
    - Better alternative would be operator input of TRACKED_BOX_COUNT before starting conveyor
      - Could also be used for resynchronization
      - Could also set default reject/divert for any boxes past PE1 rising edge
     
