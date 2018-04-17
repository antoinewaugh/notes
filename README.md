# notes

## to watch and make notes on

### functions: 

[make your functions functional](https://www.fluentcpp.com/2016/11/22/make-your-functions-functional)

### threading: 

[concurrency and parallelism with c++17 and c++20](https://www.youtube.com/watch?v=fkqVRzy4JhA)

### style: 

[the hidden rules of professional c++ code](https://www.youtube.com/watch?v=fu6N6JbPOrI)

### design: 

_API design for c++_ : Martin Reddy

### John Lakos Series

#### Large Scale c++
- [large scale c++](https://www.youtube.com/watch?v=ASPj9-4yHO0)
- [large scale c++ live-workship](https://www.safaribooksonline.com/library/view/large-scale-c-livelessonsworkshop/9780134049731/)

#### Advanced Levelization Techniques
- [Advanced Levelization Techniques (part 1 of 3)](https://www.youtube.com/watch?v=QjFpKJ8Xx78)
- [Advanced Levelization Techniques (part 2 of 3)](https://www.youtube.com/watch?v=fzFOLsFASjU)
- [Advanced Levelization Techniques (part 3 of 3)](https://www.youtube.com/watch?v=NrARQ7rHV-c)

## open source libraries

bloomberg: [bloomberg development environment](https://github.com/bloomberg/bde)

## Technique

* __underline__ : underline big ideas
* _circle_ : important words or phrases
* *questions*: write questions in margins
* _summarise_: in markdown after completing topic/chapter
* _code_ : write sample project including concepts and/or complete exercises

Reading technical books should be MORE like a conversation. Highlighting what is interesting and writing questions in the margin.

## Tidbits:

* @dascandy - 95% of my objects do not have special member definitions - should rely on underlying members to do that (having std::unique_ptr will automatically empose behaviour you may desire etc)
* john lakos: move all private methods to static free functions when they only access public accessors. better for encapsulation, also hides private implementation details from header file. 
