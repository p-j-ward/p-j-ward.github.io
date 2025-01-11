---
title: "Dunn's Single Cycle Log Analysed (part 1)"
date: 2024-03-15
tags: fpga maths
---

While searching for a way to take logs on an FPGA (without resorting to the sledgehammer of CORDIC) I stumbled upon Micheal Dunn's [Single-cycle logarithms & antilogs](https://www.edn.com/single-cycle-logarithms-antilogs/) article in EDN. His method is a clever extention of the mathematician's approximation: to take the log (base 10), just count the digits. In binary we can get the log (base 2) by "counting the digits" with a priority encoder—which outputs the position of the highest nonzero bit.

This bit-counting is pretty cheap, and gives us the _integer_ part of the log. Next is is the clever bit: we can get the _fractional_ part by left-aligning the input with a barrel shifter (by left shifting the input by the binary not of the integer part) then feeding that into a LUT (???). How on earth that works (and besides... what goes in the LUT?) is what I want to analyse here, as it falls quite neatly out of some algebra.

(TODO: add diagram here)

To work out the behaviour mathematically, first consider the priority encoder output $$P_o$$, which is the floor of the log of the input. The floor $$\lfloor a \rfloor$$ is simply $$a - \{a\}$$ where $$\{a\}$$ denotes the fractional part of $$a$$, thus for input value $$x$$:

$$ P_o = \lfloor \log_{2}x \rfloor = \log_{2}x - \{\log_{2}x\} $$

$$P_o$$ feeds straight to the output as the integer part, but before feeding it into the barrel shifter we not its bits. Why we have to do this will be apparent soon—we need to flip the signs of the terms in $$P_o$$ to get some cancellation. For now, note that the binary not $$\overline{b}$$ of the bits of a number $$b$$ means for inputs counting up ...000, ...001, ...010 we'll now be counting down ...111, ...110, ...101. That is to say $$\overline{b} = (2^N - 1) - b$$, where $$N$$ is the bit-width of $$b$$ (and thus $$2^N - 1$$ its maximum possible value).

We can now write the equation for the output of the barrel shifter $$B_o$$:

$$ B_o = x << \overline{P_o} = x << (2^N - 1 - P_o) $$

As left-shifting is exponentiation, we have:

$$ B_o = x \cdot 2^{(2^N - 1 - P_o)} = x \cdot \frac{2^{(2^N - 1)}}{2^{P_o}} $$

We can substitute in $$P_o$$ and nicely cancel out the $$x$$:

$$ B_o = x \cdot \frac{2^{(2^N - 1)}}{2^{P_o}} = x \cdot \frac{2^{(2^N - 1)}}{2^{(\log_{2}x - \{\log_{2}x\})}} = 2^{(2^N - 1)} \cdot 2^{ \{\log_{2}x\} }$$

This is why we had to bitwise not $$P_o$$ going into the barrel shifter, the not lead to the subtraction $$-P_o$$ in the equation for $$B_o$$, thus the $$\log_{2}x$$ term in $$P_o$$ cancels the $$x$$ from the input to the barrel shifter. Now just take the log of both sides, and rearrange for $$\{\log_{2}x\}$$:

$$ \log_{2}B_o = (2^N - 1) + \{\log_{2}x\}$$

Thus gives us the equation for the fractional part $$\{\log_{2}x\}$$ in terms of the barrel shifter output $$B_o$$, which is the equation that the LUT implements:

$$ \{\log_{2}x\} = \log_{2}B_o - (2^N - 1) $$

At first glance it looks like we're right back where we started, as we have to take the log of $$B_o$$, but consider the fact that—by definition—$$\{\log_{2}x\}$$ must be between zero and one, being the fractional part of $$log_{2}x$$. Thus, the value of the right hand side must also lie between zero and one. Let's rearrange it to get:

$$ \{\log_{2}x\} = \log_{2}{\frac{B_o}{ 2^{(2^N - 1)} }} = \log_{2}(B_o >> (2^N-1)) $$

Recall that $$2^N-1$$ is the maximum possible value of $$P_o$$, that is to say: it's the bit-width of the input $$x$$.

// TODO

