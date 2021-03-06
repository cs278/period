---
layout: default
title: Period objects Collection
---

# Collection

The `League\Period\Collection` is an **ordered map** that can also be used as a list of `Interval` objects.  
This class is heavily inspired by `Doctrine\Common\Collections\Collection` but also feature specific methods to deal with collection of time range objects.

~~~php
<?php

use League\Period\Collection;
use League\Period\Period;

$collection = new Sequence([
    'sports' => Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR'),
    'lunch' => Period::createFromHour('2018-05-12 12:00:00'),
]);
$collection['meeting'] = Period::createFromDuration('2018-05-12 14:00:00', '+2 HOURS');

$intersection = $collection->getIntersections();

$filterCollection = $collection->filter(function (Interval $period) use ($intersection) {
    foreach ($intersection as $interPeriod) {
        if ($interPeriod->intersects($period)) {
            return true;
        }
    }

    return false;
});

$filterCollection->contains($collection['sports']);  //true
$filterCollection->contains($collection['meeting']); //true
$filterCollection->contains($collection['lunch']);   //false
$filterCollection['meeting']->equalsTo($collection->get('meeting')); //true
~~~

## Time range related methods

### Collection::getInterval

Returns a `Period` object which represents the smallest time range containing all the `Interval` objects of the collection. If the collection is empty this methods returns `null`.

~~~php
$retval = $collection->getInterval();
//$retval is a Interval or null
~~~

### Collection::getGaps

Returns a `Collection` instance of all the gaps present in the current collection.

~~~php
$retval = $collection->getGaps();
//$retval is a Collection instance containing all the gaps from $collection
~~~

### Collection::getIntersections

Returns a `Collection` instance of all the intersections present in the current collection.

~~~php
$retval = $collection->getIntersections();
//$retval is a Collection instance containing all the intersection from $collection
~~~

## Ordered map and list methods

The class implements the following PHP interfaces:

- `ArrayAccess`
- `Countable`
- `IteratorAggregate`

In addition theses methods are also available:

### Collection::add

Append a `Interval` object to the current `Collection`.

~~~php
$collection->add(Period::createFromDay('2018-02-03'));
~~~

### Collection::contains

Tells whether the given `Interval` object is present in the current `Collection`.

~~~php
$collection = new Sequence([Period::createFromDay('2018-02-03')]);
$retval = $collection->contains(Period::createFromDay('2018-02-03')); // true
~~~

<p class="message-notice"><strong>Notice:</strong> comparison is done using <code>Interval::equalsTo</code> method</p>

### Collection::containsKey

Tells whether a `Interval` object present in the current `Collection` is attached using the `$index` key.

~~~php
$collection = new Sequence(['first' => Period::createFromDay('2018-02-03')]);
$retval = $collection->containsKey('first'); // true
~~~

<p class="message-notice"><strong>Notice:</strong> comparison is done using <code>Interval::equalsTo</code> method</p>

### Collection::clear

Remove all the `Interval` objects present in the current `Collection`.

~~~php
$collection->clear();
~~~

### Collection::filter

Filter the current `Collection` using a predicate function. If the predicate function returns `true`, the `Interval` and its related index are added in the returned `Collection` instance.

~~~php
$collection = new Sequence([
    'sports' => Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR'),
    'lunch' => Period::createFromHour('2018-05-12 12:00:00'),
    'meeting' => Period::createFromDuration('2018-05-12 14:00:00', '+2 HOURS'),
]);

$filterCollection = $collection->filter(function (Interval $period) {
    return $period->getTimestampInterval() > 60 * 60;
}); // [Period::createFromDuration('2018-05-12 14:00:00', '+2 HOURS')]
~~~

<p class="message-notice"><strong>Notice:</strong> the <code>Collection::filter</code> method uses the same arguments in the same order as <code>array_filter</code>.</p>

### Collection::get

Returns the `Interval` at the specified `$index`in the current `Collection`. If not object is found or the key is invalid, `null` is returned.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR');
$retval = $collection->get('foo'); // Interval
$retval2 = $collection->get('bar'); // null
//$retval is a Interval object or ǹull
~~~

### Collection::getKeys

Returns all the indexes/keys present in the current `Collection`.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR');
$retval = $collection->getKeys(); // ['foo']
~~~

### Collection::getValues

Returns all `Interval` objects present in the current `Collection`.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR');
$retval = $collection->getValues(); // [Interval]
~~~

### Collection::first

Returns the first `Interval` object present in the current `Collection` or `null` if the collection is
empty.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR');
$retval = $collection->first(); // Interval or null if Collection is empty
~~~

### Collection::last

Returns the last `Interval` object present in the current `Collection` or `null` if the collection is
empty.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR');
$retval = $collection->last();  // Interval or null if Collection is empty
~~~

### Collection::indexOf

Returns the index attached to the submitted `Interval` object present in the current `Collection` or `false` the `Interval` object is not found.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromDuration('2018-05-03', '+1 DAY');
$retval = $collection->indexOf(Period::createFromDay('2018-05-03')); // 'foo'
~~~

<p class="message-notice"><strong>Notice:</strong> comparison is done using <code>Interval::equalsTo</code> method</p>

### Collection::map

Apply the given function to each `Interval` in the collection and returns a new `Collection` instance containing the modified `Interval` objects. Indexes are preserved.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromDuration('2018-05-03', '+1 DAY');
$retval = $collection->map(function (Interval $period) {
    return $period->endingOn($period->getEndDate()->add(new DateInterval('P1D')));
});
$collection['foo']->getEndDate()->format('Y-m-d'); // 2018-05-4
$retval['foo']->getEndDate()->format('Y-m-d');     // 2018-05-5
~~~

### Collection::partition

Partitions this `Collection` instance in two `Collection` instances according to a predicate. Keys are preserved in the resulting collections. The first instance contains all the `Interval` objects and their related index which verify the predicate. The second instance contains the remaining objects and their indexes.

~~~php
$collection = new Sequence([
    'sports' => Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR'),
    'lunch' => Period::createFromHour('2018-05-12 12:00:00'),
    'meeting' => Period::createFromDuration('2018-05-12 14:00:00', '+2 HOURS'),
]);

$retval = $collection->partition(function (Interval $period, string $index) {
    return false !== strpos($index, 'n');
});
// [
// new Sequence([
//   'lunch' => Period::createFromHour('2018-05-12 12:00:00'),
//   'meeting' => Period::createFromDuration('2018-05-12 14:00:00', '+2 HOURS'),
//  ]),
//  new Sequence(['sports' => Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR')])
//]
~~~

<p class="message-notice"><strong>Notice:</strong> the predicate first argument is the <code>Interval</code> value and its second optional argument is its index.</p>

### Collection::remove

Removes the `Interval` object from the `Collection`. If not object was removed `false` is returned otherwise `true` is returned.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromMonth('2018-03-01');
$month = new Period('2018-03-01', '2018-04-01');
$retval = $collection->remove($month); // return true
$retval = $collection->remove(Period::createFromDay('2018-03-01')); //return false
~~~

<p class="message-notice"><strong>Notice:</strong> comparison is done using <code>Interval::equalsTo</code> method</p>

### Collection::removeIndex

Removes the `Interval` object at the specified index from the `Collection`. If not object was found `null` is returned otherwise the remove the `Interval` object is returned.

~~~php
$collection = new Sequence();
$collection['foo'] = Period::createFromMonth('2018-03-01');
$month = new Period('2018-03-01', '2018-04-01');
$retval = $collection->removeIndex('foo'); // return Period::createFromMonth('2018-03-01');
$retval = $collection->removeIndex('bar'); // return null
~~~

### Collection::set

Sets a `Interval` object in the collection at the specified key/index.

~~~php
$collection = new Sequence();
$retval = $collection->set('foo', Period::createFromWeek(2018, 21));
~~~

### Collection::slice

Extracts a slice of `$length` `Interval` objects starting at position `$offset` from the `Collection`.

Calling this method will only return the selected slice and NOT change the elements contained in the collection slice is called on.

~~~php
$collection = new Sequence([
    'sports' => Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR'),
    'lunch' => Period::createFromHour('2018-05-12 12:00:00'),
    'meeting' => Period::createFromDuration('2018-05-12 14:00:00', '+2 HOURS'),
]);
$retval = $collection->slice(0, 2);
$retval['meeting']; //returns null
~~~

<p class="message-notice"><strong>Notice:</strong> the <code>Collection::slice</code> method uses the same arguments in the same order as <code>array_slice</code></p>

### Collection::sort

Sorts the `Collection` with a user defined comparison function while maintaining the index assocation.

~~~php
$collection = new Sequence([
    'sports' => Period::createFromDuration('2018-05-12 13:30:00', '+1 HOUR'),
    'lunch' => Period::createFromHour('2018-05-12 12:00:00'),
    'meeting' => Period::createFromDuration('2018-05-12 14:00:00', '+2 HOURS'),
]);
$collection->first()->equalsTo($collection['sports']);
$collection->las()->equalsTo($collection['meeting']);
$collection->sort(function (Interval $period1, Interval $period2) {
    return $period2->compareDuration($period1);
});
$collection->first()->equalsTo($collection['lunch']);
$collection->las()->equalsTo($collection['meeting']);
~~~

<p class="message-notice"><strong>Notice:</strong> the <code>Collection::sort</code> method uses the same arguments as <code>uasort</code>.</p>