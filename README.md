# Generic Additive Synthesis

Julian Rohrhuber and Juan Sebastián Lach Lau.

A generalisation of additive synthesis, as described in our forthcoming article *Generic Additive Synthesis. Hints from the Early Foundational Crisis in Mathematics for Experiments in Sound Ontology*

Here we provide some examples from the article and will add more in the future.


A few simple examples, using [SuperCollider](https://github.com/supercollider/supercollider):

```

// simple case of concatenative frequency modulation
(
Ndef(\g, {
	var combinator = { |a, b| a <> b };
	var c = { |i| LFTri.ar(1 / i, 1 / i).range(-1, 1).max(0) };
	var g = { |x, i|
		SinOsc.ar(i * 40, x * 3)
	};
	var n = (1..12); // number of operands
	n.inject(0, { |x, i| // inject is also known as left fold. Base case is 0 here
		g.(x, i) * c.(i) + (x * (1 - c.(i)))
	}) * 0.1
}).play;
)

// generic additive synthesis with a sine basis and addition
(
Ndef(\g, {

	var combinator = { |a, b| a + b }; // just binary sum here
	var c = { |i| 1 / ((i % 7) + (i % 8)  + (i % 11) + 1) };
	var g = { |i|
		SinOsc.ar(110 * i) * c.(i); // basis function ("dimension")
	};
	var z = (1..30); // number of operands
	var set = z.collect { |i| g.(i) }; // sequence
	 // combine and scale output:
	set.reduce(combinator) ! 2 * (1 / z.size)

}).play;
)

// generic additive synthesis with a more complicated basis and a product combinator
(
Ndef(\g, {
	var combinator = { |a, b| a * b };
	var c = { |i| 8 / i };
	var g = { |i|
		var cn = c.(i);
		var y1 = SinOsc.ar(120 * i, SinOsc.ar(cn * 10 * i) * (1/i));
		var y2 = LFPulse.kr(cn, 0, SinOsc.ar(cn * i, i, 0.2, 0.3));
		y1 * y2 * cn + 1
	};
	var n = (1..12); // number of operands
	var set = n.collect { |i| g.(i) }; // sequence
	LeakDC.ar(set.reduce(combinator) * (0.01 / n.size)).tanh ! 2
}).play;
)
```


Another example, using the [Steno](https://github.com/telephon/Steno) embedded language:

```




t = Steno.push(8);

(
t.filter(\g, { |in, envir|
	var r = \rotate.kr(0);
	in = in.collect { |x| PanAz.ar(in.size, x, 2*r/in.size) }.sum;
	SinOsc.ar((100 / (\index.kr + 1)), in * 2)
});
t.filter('.', { |in|
	Splay.ar(in)
})
);


--ggggg.

t.setGlobal(\index, { |i| i + 1 });

```

