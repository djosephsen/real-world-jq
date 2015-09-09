## The problem:
Given a full dump of output from ```aws ec2 describe-instances```, select and
return the full instance object for every instance that contains a given substring. 

## The Winner: 
```
jq '.[][].Instances[] | select((.Tags[]?.Value?) | contains("TAG")?)' 
```
... where ```TAG``` is the tag you want to match

## Gotchas: 
This was actually a really hairy problem. I pretty quickly solved an exact
string match, eg: 

```
jq '.[][].Instances[] | select(.Tags[]?.Value? == "MY-EXACT-TAG")'
```

But I kept on getting an error trying to use ```contains()``` to match a
substring. It looked like this: 

``` 
jq: error (at <stdin>:1): null (null) and string ("DJ") cannot have their containment checked
```

I couldn't figure out for the life of me where that null input string was
coming from, especially since I was protecting .Tags[] AND .Value with the
question mark operator, which *should* make this ignore objects that don't have
either Tags or Tags.Value.

And actually, that *was* working. But for objects with empty Tags or
Tags.Value, the ? operator silently provided a null value (which is totally
what it's supposed to do), so effectively, for the instances that are missing
tags, the select statement wound up looking like this:

```
select((null) | contains("TAG"))'
```

## Solution: 

The secret sauce was that I also had to protect the *select* function with a
```?``` operator. Basically, I needed this: 

```
select()?
```

That way, if the select statement exits null, that object is silently ignored
and processing goes on as normal. 
