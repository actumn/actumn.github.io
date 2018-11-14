---
layout: posts
title:  "Go의 Concurrency"
date:   2018-11-11
categories: cs
---
- [Learning Go's Concurrency](https://medium.com/@trevor4e/learning-gos-concurrency-through-illustrations-8c4aff603b3)  
  
```
func main() {
 theMine := [5]string{“rock”, “ore”, “ore”, “rock”, “ore”}
 foundOre := finder(theMine)
 minedOre := miner(foundOre)
 smelter(minedOre)
}
output:
From Finder: [ore ore ore]
From Miner: [minedOre minedOre minedOre]
From Smelter: [smeltedOre smeltedOre smeltedOre]
```
위와 같은 싱글스레드 프로그램이 있다.  
멀티스레드로 동작시키기 위해선?

## Go routines
  
```
func main() {
 theMine := [5]string{“rock”, “ore”, “ore”, “rock”, “ore”}
 go finder1(theMine)
 go finder2(theMine)
 <-time.After(time.Second * 5) //you can ignore this for now
}
output: 
Finder 1 found ore!
Finder 2 found ore!
Finder 1 found ore!
Finder 1 found ore!
Finder 2 found ore!
Finder 2 found ore!
```
오, concurrency한 동작  
go rountine들이 상호작용을 해야할 땐?

## Channels
채널은 go routine들이 상호작용하기 위한 파이프  
```
myFirstChannel := make(chan string)
myFirstChannel <- "hello" // Send
myVariable := <- myFirstChannel // Receive
```
화살표(<-)를 통해 send, receive를 수행.  
위의 예제코드는 다시 아래와 같이 변경  
```
func main() {
 theMine := [5]string{“ore1”, “ore2”, “ore3”}
 oreChan := make(chan string)
 // Finder
 go func(mine [5]string) {
  for _, item := range mine {
   oreChan <- item //send
  }
 }(theMine)
 // Ore Breaker
 go func() {
  for i := 0; i < 3; i++ {
   foundOre := <-oreChan //receive
   fmt.Println(“Miner: Received “ + foundOre + “ from finder”)
  }
 }()
 <-time.After(time.Second * 5) // Again, ignore this for now
}
output:
Miner: Received ore1 from finder
Miner: Received ore2 from finder
Miner: Received ore3 from finder
```

## Channel Blocking  
채널은 go routine을 블로킹한다.  
```
myFirstChannel <- "hello" // Send
```
go routine이 채널에 데이터를 send하면,  
전송한 go routine은 다른 go routine이 receive 할때 까지 블로킹
```
myVariable := <- myFirstChannel // Receive
```
채널에 send할 때 처럼,  
go routine은 receive 시에 채널에 send된 데이터가 없으면 받을때까지 블로킹  
  
채널에는 unbuffered, buffered로 나뉜다.
### Unbuffered Channels
Unbuffered 채널에는 한 번에 1 데이터만  

### Buffered Channels
concurrent program에선 timing이 완벽하지 않다.  
채널에 데이터를 send할 go routine이 receive할 go routine보다 더 빨리 돌 수도 있는 것.  
이 때를 위한 Buffered 채널
```
bufferedChan := make(chan string, 3)
```
Buffered 채널은 unbuffered 채널과 비숫하게 동작.  
다만 receive 되기까지 여러개의 데이터를 eend 받을 수 있다.
```
bufferedChan := make(chan string, 3)
go func() {
 bufferedChan <- "first"
 fmt.Println("Sent 1st")
 bufferedChan <- "second"
 fmt.Println("Sent 2nd")
 bufferedChan <- "third"
 fmt.Println("Sent 3rd")
}()
<-time.After(time.Second * 1)
go func() {
 firstRead := <- bufferedChan
 fmt.Println("Receiving..")
 fmt.Println(firstRead)
 secondRead := <- bufferedChan
 fmt.Println(secondRead)
 thirdRead := <- bufferedChan
 fmt.Println(thirdRead)
}()
output: 
Sent 1st
Sent 2nd
Sent 3rd
Receiving..
first
second
third
```

## 최종적으로
```
func main(){
  theMine := [5]string{"rock", "ore", "ore", "rock", "ore"}
  oreChannel := make(chan string)
  minedOreChan := make(chan string)
  // Finder
  go func(mine [5]string) {
   for _, item := range mine {
    if item == "ore" {
     oreChannel <- item //send item on oreChannel
    }
   }
  }(theMine)
  // Ore Breaker
  go func() {
   for i := 0; i < 3; i++ {
    foundOre := <-oreChannel //read from oreChannel
    fmt.Println("From Finder: ", foundOre)
    minedOreChan <- "minedOre" //send to minedOreChan
   }
  }()
  // Smelter
  go func() {
   for i := 0; i < 3; i++ {
    minedOre := <-minedOreChan //read from minedOreChan
    fmt.Println("From Miner: ", minedOre)
    fmt.Println("From Smelter: Ore is smelted")
   }
  }()
  <-time.After(time.Second * 5) // Again, you can ignore this
}
output:
From Finder:  ore
From Finder:  ore
From Miner:  minedOre
From Smelter: Ore is smelted
From Miner:  minedOre
From Smelter: Ore is smelted
From Finder:  ore
From Miner:  minedOre
From Smelter: Ore is smelted
```

아, 메인함수도 go routine이라는 듯

### range over a channel
채널에 얼마나 send될 지 모를 경우
``` 
 // Ore Breaker
 go func() {
  for foundOre := range oreChan {
   fmt.Println(“Miner: Received “ + foundOre + “ from finder”)
  }
 }()
```

### non-blocking read on a channel
channel의 non-blocking read를 위한 테크닉.  
select case를 쓴다.  
```
myChan := make(chan string)
 
go func(){
 myChan <- “Message!”
}()
 
select {
 case msg := <- myChan:
  fmt.Println(msg)
 default:
  fmt.Println(“No Msg”)
}
<-time.After(time.Second * 1)
select {
 case msg := <- myChan:
  fmt.Println(msg)
 default:
  fmt.Println(“No Msg”)
}

output:
No Msg
Message!
```
non-blocking send도 있다.
```
select {
 case myChan <- “message”:
  fmt.Println(“sent the message”)
 default:
  fmt.Println(“no message sent”)
}
```

### Parallelism은?
Parallelism을 위해선 [GOMAXPROCS를 건드려야 한다.](https://www.ardanlabs.com/blog/2014/01/concurrency-goroutines-and-gomaxprocs.html)
