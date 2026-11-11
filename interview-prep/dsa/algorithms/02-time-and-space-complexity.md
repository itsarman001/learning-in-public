# Time and Space Complexity

The absolute running time of an algorithm cannot be predicted accurately because it depends on several factors:

- **Programming language:** The language used to implement the algorithm.
    
- **Hardware:** The machine on which the program runs.
    
- **Background processes:** Other programs running simultaneously.
    
- **Operating system:** The efficiency and quality of the operating system.
    

Instead of measuring the actual execution time, we evaluate an algorithm's performance in terms of its **input size**.

## 1. Types of Complexity

### 1.1 Time Complexity

The amount of time an algorithm takes to execute as a function of its input size.

### 1.2 Space Complexity

The amount of memory an algorithm requires to execute as a function of its input size.

Evaluating complexity based on input size makes the analysis **machine-independent** and allows us to compare algorithms more consistently.

## 2. Choosing an Algorithm

There is no single solution that works best in every situation.

It is important to understand multiple approaches to solving a problem and choose the most suitable one based on the given constraints, such as execution time and memory requirements.

## 3. How Complexity Is Represented

Algorithm complexity is represented using **Asymptotic Notations**.

Asymptotic notations are mathematical tools used to describe the growth of an algorithm's time or space requirements as its input size increases.

### 3.1 Types of Asymptotic Notations

|Notation|Name|Represents|
|---|---|---|
|**O(n)**|Big-O|Upper bound (commonly used to describe worst-case complexity)|
|**Ω(n)**|Omega|Lower bound (commonly used to describe best-case complexity)|
|**Θ(n)**|Theta|Tight bound (when upper and lower bounds match)|

These notations help us understand how an algorithm scales without depending on the specific machine or programming language.