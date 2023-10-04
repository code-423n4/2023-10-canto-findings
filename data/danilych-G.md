# [FM-03] Redundant Division and Multiplication Operations
## Description
  In the contract code, there are instances of redundant mathematical operations involving division and multiplication. Specifically, on lines  `L51-52`, `L96-97`, `L208-209`, and `L238-239`, a division operation is followed immediately by a multiplication with the same value. These operations are redundant and not efficient in terms of gas consumption.  
## Recommendations
  It is recommended to refactor the code to eliminate the unnecessary division and multiplication operations. Instead, directly use the resulting value from the division where applicable to improve code efficiency.