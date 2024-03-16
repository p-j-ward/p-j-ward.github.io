---
title: "Dunn's Single Cycle Log Analysed (part 1)"
date: 2024-03-15
tags: fpga maths
---

While searching for a way to take logs on an FPGA (without resorting to the sledgehammer of CORDIC) I stumbled upon Micheal Dunn's [Single-cycle logarithms & antilogs](https://www.edn.com/single-cycle-logarithms-antilogs/) article in EDN. His method is a clever extention of the mathematician's approximation: to take the log (base 10), just count the digits. In binary we can get the log (base 2) by "counting the digits" with a priority encoder—which outputs the position of the highest nonzero bit.

This bit-counting is pretty cheap, and gives us the integer part of the log. Next is is the clever bit: we can get the fractional part by left-aligning the input with a barrel shifter (by left shifting the input by the binary not of the integer part) then feeding that into a LUT (???). How on earth that works (and besides... what goes in the LUT?) is what I want to analyse here, as it falls quite neatly out of some simple algebra, with a few tricks.

(TODO: add diagram here)

To work out the behaviour mathematically, first consider the priority encoder output $$P_o$$, which is the floor of the log (base 2). The floor $$\lfloor a \rfloor$$ is simply $$a - \{a\}$$ where $$\{a\}$$ denotes the fractional part of $$a$$, thus for input value $$x$$:

$$ P_o = \lfloor \log_{2}x \rfloor = \log_{2}x - \{\log_{2}x\} $$

$$P_o$$ feeds straight to the output as the integer part of the log, but before feeding it into the barrel shifter we not it's bits. Why we have to do this will be apparent soon—we need to flip the signs of the terms in $$P_o$$ to get some cancellation. For now, note that the binary not $$\bar b$$ of the bits of a number $$b$$ means for inputs counting up $$...000$$, $$...001$$, $$...010$$ we'll now be counting down $$...111$$, $$...110$$, $$...101$$. That is to say $$\bar b = (2^N - 1) - b$$, where $$N$$ is $$b$$'s width in bits (and thus $$2^N - 1$$) its maximum possible value).

We can now write the equation for the output of the barrel shifter $$B_o$$:

$$B_o = x << \bar P_o = $$
