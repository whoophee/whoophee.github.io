---
layout: post
title:  "Bloom Filter"
date:   2018-10-12
excerpt: "Space efficient probabilistic data structure."
tags: [data structures, probabilistic data structures]
comments: true
belongsto: posts
mathjax: true
---
"A Bloom filter is a space-efficient probabilistic data structure, conceived by Burton Howard Bloom in 1970, that is used to test whether an element is a member of a set." - Wikipedia

Several weeks of rummaging through data structures introduced me to the Bloom Filter. And it is an absolute thing of beauty. I've implemented/explained a very rudimentary version of said data structure to better understand it.

## Basic implementation
The data structure requires a bit-array and a few different hash functions in its implementation.

For this explanation, consider a 20 bit-array with all bits set to 0. The hash functions used are MurmurHash3 and fnv.

{% highlight python %}
import mmh3
import fnv
bit_array = [0] * 20
hashfn_1 = mmh3.hash
hashfn_2 = fnv.hash
{% endhighlight %}

<table class="bloom-table">
	<tr>{% for idx in (0..19) %}<th>{{ idx }}</th>{% endfor %}</tr>
	<tr>
		{% for idx in (0..19) %}
			{% assign i = idx | downcase | append: "#" | prepend: "#" %}
			{% if " " contains i %}<td class="bloom-add-old">1</td>
			{% elsif " " contains i %}<td class="bloom-add-new">1</td>
			{% else %}<td>0</td>
			{% endif %}
		{% endfor %}
	</tr>
</table>

### Insertion
Inserting a string involves hashing the string with each of the hash functions and setting the corresponding bits of the array.
{% highlight python %}
string1 = b"Today is a Sunday"
str1_mmh = hashfn_1(string1) % 20 # 0
str1_fnv = hashfn_2(string1) % 20 # 15
bit_array[ str1_mmh ] = 1
bit_array[ str1_fnv ] = 1
{% endhighlight %}

Newly set bits are marked in green.

<table class="bloom-table">
	<tr>{% for idx in (0..19) %}<th>{{ idx }}</th>{% endfor %}</tr>
	<tr>
		{% for idx in (0..19) %}
			{% assign i = idx | downcase | append: "#" | prepend: "#" %}
			{% if " " contains i %}<td class="bloom-add-old">1</td>
			{% elsif "#0#15#" contains i %}<td class="bloom-add-new">1</td>
			{% else %}<td>0</td>
			{% endif %}
		{% endfor %}
	</tr>
</table>

Similarly, setting the corresponding hashed bits of the second string results in,
{% highlight python %}
>>> string2 = b"Tomorrow is a Monday"
>>> hashfn_1(string2) % 20
18
>>> hashfn_2(string2) % 20
19
{% endhighlight %}

Bits that were set in previous transactions are marked in yellow.

<table class="bloom-table">
	<tr>{% for idx in (0..19) %}<th>{{ idx }}</th>{% endfor %}</tr>
	<tr>
		{% for idx in (0..19) %}
			{% assign i = idx | downcase | append: "#" | prepend: "#" %}
			{% if "#0#15#" contains i %}<td class="bloom-add-old">1</td>
			{% elsif "#18#19#" contains i %}<td class="bloom-add-new">1</td>
			{% else %}<td>0</td>
			{% endif %}
		{% endfor %}
	</tr>
</table>

At this point, it becomes evident that the structure does not require an in depth understanding of the hash functions used. However, it is also obvious that most of the heavy lifting during insertion and checking, takes place during hashing. Thus, the hash functions need to be as fast as possible (and cryptographically strong functions are not very good choices).
### Membership of elements

Hereon, testing the candidacy of a string seems obvious enough,

{% highlight python %}
def insert_string(samplestr):
	bit_array[hashfn_1(samplestr) % 20] = 1
	bit_array[hashfn_2(samplestr) % 20] = 1

def check_string(samplestr):
	if bit_array[hashfn_1(samplestr) % 20] and bit_array[hashfn_2(samplestr) % 20]:
		return True
	return False
{% endhighlight %}

Here, check string returns true if all corresponding bits are set in the bit-array.

For the string "Today is a Sunday", the 0<sup>th</sup> and 15<sup>th</sup> bits need to be set.

<table class="bloom-table">
	<tr>{% for idx in (0..19) %}<th>{{ idx }}</th>{% endfor %}</tr>
	<tr>
		{% for idx in (0..19) %}
			{% assign i = idx | downcase | append: "#" | prepend: "#" %}
			{% if "#0#15#18#19#" contains i %}<td class="bloom-add-old">1</td>
			{% elsif " " contains i %}<td class="bloom-add-new">1</td>
			{% else %}<td>0</td>
			{% endif %}
		{% endfor %}
	</tr>
	<tr>
		{% for idx in (0..19) %}
		<td class="bloom-arrow">
			{% assign i = idx | downcase | append: "#" | prepend: "#" %}
			{% if "#0#15#" contains i %}<i class="fa fa-arrow-up" aria-hidden="true"></i>
			{% else %}
			{% endif %}
		</td>
		{% endfor %}
	</tr>
</table>

And it's as simple as that.

### Wrapper class
I wrapped the described functionalities into a single class.

{% highlight python %}
import mmh3
import fnv

class BloomFilter:
	def __init__(self, filter_size = 20, hash_functions = [mmh3.hash, fnv.hash]):
		self.bitarray_size = filter_size
		self.bitarray = [0] * filter_size
		self.hash_functions = hash_functions
	def check(self, str_input):
		for hash_fn in self.hash_functions:
			hash_idx = hash_fn(str_input) % self.bitarray_size
			if not self.bitarray[hash_idx]:
				return False
		return True
	def insert(self, str_input):
		for hash_fn in self.hash_functions:
			hash_idx = hash_fn(str_input) % self.bitarray_size
			self.bitarray[hash_idx] = True
{% endhighlight %}

## Further analysis

In its quest to be space efficient, it fails at capturing the uniqueness of an element.

Consider the following,

{% highlight python %}
>>> hashfn_1(b'e')
19
>>> hashfn_2(b'e')
0
{% endhighlight %}

<table class="bloom-table">
	<tr>{% for idx in (0..19) %}<th>{{ idx }}</th>{% endfor %}</tr>
	<tr>
		{% for idx in (0..19) %}
			{% assign i = idx | downcase | append: "#" | prepend: "#" %}
			{% if "#0#15#18#19#" contains i %}<td class="bloom-add-old">1</td>
			{% elsif " " contains i %}<td class="bloom-add-new">1</td>
			{% else %}<td>0</td>
			{% endif %}
		{% endfor %}
	</tr>
	<tr>
		{% for idx in (0..19) %}
		<td class="bloom-arrow">
			{% assign i = idx | downcase | append: "#" | prepend: "#" %}
			{% if "#0#19#" contains i %}<i class="fa fa-arrow-up" aria-hidden="true"></i>
			{% else %}
			{% endif %}
		</td>
		{% endfor %}
	</tr>
</table>

This gives rise to a false membership positive. Since such a situation is inevitable, it needs to be further analysed to arrive at optimal conditions.

### False positive rate and optimal hashes <sup>[\[1\]](#cite1)</sup>

Let the number of hash functions used be *k* and the size of the bit-array be *m*. Assuming the bits occur uniformly, the probability that a bit is not set would be $$ 1 - \frac{1}{m} $$.

With *k* hash functions, the probability that it remains unset would be $$\left(1-\frac{1}{m}\right)^k $$.

After insertion of *n* elements this probability becomes $$\left(1-\frac{1}{m}\right)^{nk} $$

Thus, the probability that the bit is already set is $$\left[1-\left(1-\frac{1}{m}\right)^{nk}\right] $$

The probability that all k bits of the current element is already set is thus,
<center>$$p = \left[1-\left(1-\frac{1}{m}\right)^{nk}\right]^k \approx \left(1 - e^{-kn/m}\right)^k $$</center>

This probability of false positive is minimized when <sup>[\[2\]](#cite2)</sup> <center>$$k = \frac{m}{n}ln2$$</center>

Plugging in this value results in an optimal value <sup>[\[3\]](#cite3)</sup> <center>$$m = -\frac{n * lnp}{(ln 2)^2} $$</center>

This goes to show that, the optimal number of hash functions is decided by the room for false membership rate and the ballpark  number of elements that might be inserted in the filter.

So yeah, that pretty much sums up my understanding of the Bloom Filter.

## Examples

It makes more sense that I link you to the [wiki](http://en.wikipedia.org/wiki/Bloom_filter#Examples) instead of copying whatever it says.

## Reference links
[1] [Wikipedia Bloom Filter : Probability of false positives](https://en.wikipedia.org/wiki/Bloom_filter#Probability_of_false_positives){:id="cite1"}

[2] [UC Berkeley CS 170: Efficient Algorithms and Interactable Problems](https://people.eecs.berkeley.edu/~daw/teaching/cs170-s03/Notes/lecture10.pdf){:id="cite2"}

[3] [Wikipedia Bloom Filter : Optimal number of hash functions](https://en.wikipedia.org/wiki/Bloom_filter#Optimal_number_of_hash_functions){:id="cite3"}
