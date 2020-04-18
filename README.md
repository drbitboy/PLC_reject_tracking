    Cf. [https://www.plctalk.net/qanda/showthread.php?t=124552](https://www.plctalk.net/qanda/showthread.php?t=124552)
    
    Process
    =======
    Boxes on a conveyor; two PhotoEye stations (PE1; PE2)
    Each box's status judged as [reject] or [okay] when it generates a rising edge at upstream PE1
    When each box later generates a rising edge at downstream PE2:
    - divert if box status from PE1 was [reject]
    - do not divert if box status from PE1 was [okay]
    
    Implementation summary
    ====================
    Bit FIFO/BSR with bit=1 for [reject], bit=0 for [okay]
    FIFO size >> number of boxes from PE1 to before PE2
    Always push 0-valued bit (status [okay]) at front of FIFO
    Keep track of count of boxes in index PAST_PE1_NOT_PE2
    At PE2 rising edge
    - Pop reject/okay bit
    - Decrement index 
    At PE1 rising edge
    - Overwrite FIFO[index] 1-valued bit if current judgment is reject
    - Increment index
    N.B. Assumptions
    - Adequate space between boxes, so 1 rising edge per box at PE1 and at PE2
    - No boxed added or subtracted between PE1 and PE2 (or coffee cup blocks PE)
    
    Initialization
    =========
    Set diverted state to zero
    - Alternate:  detect diverter state
    Set FIFO and index to zero
    - Better would be operator input of PAST_PE1_NOT_PE2 index
      - Could also be used for resynchronization
      - Could also set default reject/divert for any boxes past PE1 rising edge
