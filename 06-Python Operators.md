# 06.Python Operators

**Python Operators**

Operators are used to performing operations on variables and values.

In the example below, we use the + operator to add together two values:

**Example**

**print(10 + 5)**

Python divides the operators into the following groups:

- Arithmetic operators

- Assignment operators

- Comparison operators

- Logical operators

- Identity operators

- Membership operators

- Bitwise operators

**Python Arithmetic Operators**

Arithmetic operators are used with numeric values to perform common
mathematical operations:

  ------------------------------------------------------------------------
  **Operator**          **Name**                       **Example**
  --------------------- ------------------------------ -------------------
  \+                    Addition                       x + y

  \-                    Subtraction                    x - y

  \*                    Multiplication                 x \* y

  /                     Division                       x / y

  \%                    Modulus                        x % y

  \*\*                  Exponentiation                 x \*\* y

  //                    Floor division                 x // y
  ------------------------------------------------------------------------

**Python Assignment Operators**

Assignment operators are used to assigning values to variables:

  -----------------------------------------------------------------------
  Operator               Example                Same As
  ---------------------- ---------------------- -------------------------
  =                      x = 5                  x = 5

  +=                     x += 3                 x = x + 3

  -=                     x -= 3                 x = x - 3

  \*=                    x \*= 3                x = x \* 3

  /=                     x /= 3                 x = x / 3

  %=                     x %= 3                 x = x % 3

  //=                    x //= 3                x = x // 3

  \*\*=                  x \*\*= 3              x = x \*\* 3

                                                

  &=                     x &= 3                 x = x & 3

  \|=                    x \|= 3                x = x \| 3

  \^=                    x \^= 3                x = x \^ 3

  \>\>=                  x \>\>= 3              x = x \>\> 3

  \<\<=                  x \<\<= 3              x = x \<\< 3
  -----------------------------------------------------------------------

**Python Identity Operators**

Identity operators are used to comparing the objects, not if they are
equal, but if they are actually the same object, with the same memory
location:

  ------------------------------------------------------------------------------
  **Operator**   **Description**                                   **Example**
  -------------- ------------------------------------------------- -------------
  is             Returns True if both variables are the same       x is y
                 object                                            

  Is not         Returns True if both variables are not the same   x is not y
                 object                                            
  ------------------------------------------------------------------------------

**Python Membership Operators**

Membership operators are used to testing if a sequence is presented in
an object:

  -----------------------------------------------------------------------------------
  **Operator**   **Description**                                        **Example**
  -------------- ------------------------------------------------------ -------------
  in             Returns True if a sequence with the specified value is x in y
                 present in the object                                  

  not in         Returns True if a sequence with the specified value is x not in y
                 not present in the object                              
  -----------------------------------------------------------------------------------

**Python Comparison Operators**

Comparison operators are used to comparing two values:

  ------------------------------------------------------------------------
  **Operator**    **Name**                        **Example**
  --------------- ------------------------------- ------------------------
  ==              Equal                           x == y

  !=              Not equal                       x != y

  \>              Greater than                    x \> y

  \<              Less than                       x \< y

  \>=             Greater than or equal to        or equal to x \>= y

  \<=             Less than or equal to           x \<= y
  ------------------------------------------------------------------------

**Python Logical Operators**

Logical operators are used to combining conditional statements:

  ----------------------------------------------------------------------------
  **Operator**   **Description**                            **Example**
  -------------- ------------------------------------------ ------------------
  and            Returns True if both statements are true   x \< 5 and x \< 10

  or             Returns True if one of the statements is   x \< 5 or x \< 4
                 true                                       

  not            Reverse the result, returns False if the   not(x \< 5 and x
                 result is true                             \< 10)
  ----------------------------------------------------------------------------

**Python Bitwise Operators**

Bitwise operators are used to comparing (binary) numbers:

  -----------------------------------------------------------------------------
  **Operator**   **Name**   **Description**
  -------------- ---------- ---------------------------------------------------
  &              AND        Sets each bit to 1 if both bits are 1

  \|             OR         Sets each bit to 1 if one of two bits is 1

  \^             XOR        Sets each bit to 1 if only one of two bits is 1

  \~             NOT        Inverts all the bits

  \<\<           Zero fill  Shift left by pushing zeros in from the right and
                 left shift let the leftmost bits fall off

  \>\>           Signed     Shift right by pushing copies of the leftmost bit
                 right      in from the left, and let the rightmost bits fall
                 shift      off
  -----------------------------------------------------------------------------
