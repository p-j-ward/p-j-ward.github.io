---
title: "Dunn's Single Cycle Log Analysed (part 1)"
date: 2024-03-15
tags: fpga maths
---

While searching for a way to take logs on an FPGA (without resorting to the sledgehammer of CORDIC) I stumbled upon Micheal Dunn's [Single-cycle logarithms & antilogs](https://www.edn.com/single-cycle-logarithms-antilogs/) article in EDN. His method relies first on the mathematician's approximation: to take the log (base 10), just count the digits. In binary we can get the log (base 2) by "counting the digits" with a priority encoder—which returns the position of the highest nonzero bit.

// TODO
