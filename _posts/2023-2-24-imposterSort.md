---
layout: post
title: "imposter sort"
---
hi. i have invented a new sorting algorithm, that I am naming "imposter sort".

the algorithm is quite simple, but don't let that distract from it's power: it is the only comparison based algorithm that achieves O(n) time. for comparison (heh), [quick sort](https://en.wikipedia.org/wiki/Quicksort) - one of the most popular sorting algorithms for its speed - has a best case time complexity is O(n log n): much slower. 

in fact, O(n log n) *is* the [fastest any comparison based algorithm](https://stackoverflow.com/a/7155425) can ever do (with edge case exceptions). how did i do it? 

## the algorithm

first, lets try to understand our goal. 

given an array of any length and with potentially random elements, we want to transform the array in a way such that for every element, it's preceding element's value is comparitively lesser than the current element's value. 

simple enough! in order to truly understand the algorithm, let's reframe this a little bit.

imagine, if you will, that we have a group of people, and we line them up one by one. we need to verify that each person is safe, and not a faker. now the thing is, each person knows who to stand next to, and any pretender in the group wouldn't know. additionally, each person has an identification number, which dictates who they should be standing next to.

so what we can do is go through each member, and check their identification to see if the person before them truly does go before. if the person is out of place, that means either they made a mistake, or they is an fraud. in either case, we should treat them with suspicion and get them out of the line. can't be too careful.

however... i'll just vent to you, those imposters are too suspicious! i'd personally vote to kick them off the group all together, and for the purposes of the algorithm, that's what we'll do. after that, we now have a line of people who are clear! 

note that we only had to go through all the people once, guaranteeing O(n) time complexity. that's fast! 


## code

here's the algorithm, implemented in javascript:
```
function imposterSort(array) {
	if (array.length == 0) return [];
	const newArray = [array[0]];
	for (let i = 1; i < array.length; i++) {
		if (array[i] < newArray[newArray.length - 1])
			continue;
		newArray.push(array[i]);
	}
	return newArray;
}
```

let's walk through it.

```
if (array.length == 0) return [];
const newArray = [array[0]];
```
the algorithm will error if there is nothing in the array, so it just exits immediately. we additionally initialize `newArray` to an array with the first element of the input.

```
for (let i = 1; i < array.length; i++)
```
we start at `1` because we can make a well educated guess that the first element is sorted.

```
if (array[i] < newArray[newArray.length - 1])
	continue;
newArray.push(array[i]);
```
here's the magic: the end of `newArray` is the 'last' clear element. if the current element is less than that, then the current element is an imposter, so we ignore it. otherwise, our task is to clear it, which we do by simple pushing the element into `newArray`. 

let's test it!

### benchmark!!

we're going to use [jsbench](https://jsbench.me/) for this.

first i'll set the "Setup JavaScript" code:
```
function imposterSort(array) {
	if (array.length == 0) return [];
	const newArray = [array[0]];
	for (let i = 1; i < array.length; i++) {
		if (array[i] < newArray[newArray.length - 1])
		continue;
		newArray.push(array[i]);
	}
	return newArray;
}

// array randomizer
const array = [];
for (let i = 0; i < 100000; i++) {
	array.push(Math.floor(Math.random() * 100));
}
```

then the two test cases:
```
// standard sort
array.sort((a, b) => a - b)
```

```
// imposter sort
imposterSort(array);
```
(note, the javascript runtime V8, used in most chromium based browsers and node.js apps, at the time of writing this, uses [Timesort](https://en.wikipedia.org/wiki/Timsort) for it's array.sort method) 

running the test yeilds these results on my computer consistently. 

![jsbench results, testing javascript's built-in array sorting method against imposter sort](/assets/images/imposterSort-test1.png)

the natively implemented sorting algorithm performs *96.91%* slower. i'm quite proud of that, tbh.

## improvement

one flaw with the algorithm is that it creates a new array, which can be quite slow. we can fix this is by doing the sort "in-place".

here's what I came up with:
```
function imposterSort_inPlace(array) {
	let i = 1, j = 0;
	for (; i < array.length; i++) {
		if (array[i] < array[j])
			continue;
		j++;
		array[j] = array[i];
	}
	for (let n = 0; n < i - j - 1; n++) {
		array.pop();
	}
	return array;
}
```

the most important difference is the comparison:
```
if (array[i] < array[j])
	continue;
```

if this looks familiar, good. the `j` variable is emulating the length of `newArray` from earlier. 

we also do this:
```
for (let n = 0; n < i - j - 1; n++) {
	array.pop();
}
```
to remove any left over elements in the array. 

adding this as a test case yeilds these results:

![jsbench results, testing javascript's built-in array sorting method against imposter sort and in-place imposter sort](/assets/images/imposterSort-test2.png)

suddenly the normal imposterSort is *98.9%* slower. pretty good, imo!


# conclusion

the imposter sort is ridiculously fast for the power it provides. i hope you consider using it in your next project. 


