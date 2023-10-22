---
title: Visual debugging with Swift Charts
date: 2023-10-20 00:00:01 Z
layout: post
image: swift-charts-header.png
description: When you are working with a lot of data, it is hard to see what is going
  on. Swift Charts can help you visualise your data and spot issues.
---

<span class="dropcap">W</span>hen I decided to implement the algorithm Soroush Khanlou so brilliantly described in his [Elevated Swift talk](https://www.youtube.com/watch?v=-v1huP4RBgI) I didn’t expect that the hardest part would be not the algorithm for the elevator (or lift, as we call it [here](https://en.wikipedia.org/wiki/United_Kingdom)), but rather the one for generating the traffic.

Let’s take a step back and describe what we are trying to achieve. We have an office building with a few lifts going up and down the floors. As a typical office building we want to simulate a realistic flow of people - more during the week, less on weekends; more arriving in the morning, less so at night. We also want to keep the randomness element. And let’s not forget the lunch break with lots of people going out and back in mumbling about the queue at the lifts.

When we take all this into consideration, our next step is to decide how often we run the algorithms - both for the lifts and for the traffic flow. As we want to ideally animate the lifts at some point, we need a loop running at a set interval, ideally a short one. We also need fake in-game time or otherwise our game will only work in real time and everything will take forever. This will also help us implement controls to slow and fasten the in-game time. After some experimentation, I arrived the following:
- We have a timer that "ticks" at a set interval.
- Every “tick” equals 10 seconds in-game time.
- The initial “tick” interval is 0.5 seconds, with controls for changing it down to 0.01 (less is faster).

Some sloppy SwiftUI designing later, and we have our first running version. We can see the three lifts going up and down. There are people arriving at the ground floor. The lifts seem to be working well.

<img src="{{ '/assets/img/lifts-moving.gif' | prepend: site.baseurl }}" style="width: 300px; border-width: 1px; border-color: #b20600; border-style: double;" alt="">

I showed it to a friend at the office and we decided to speed it up and see what happens. The first thing we noticed was that whenever new people called a lift, all three lifts would head their way. The algorithm was not truly multi-lift.

The second thing we spotted we only saw because we decided to keep the game running for a while whilst we were talking. The problem was that there were quite a few people arriving on Sunday. The people generation algorithm should have prevented this from happening. The problem became apparent when we opened the stats screen. The data for every day was the same.

<img src="{{ '/assets/img/swift-charts-header.png' | prepend: site.baseurl }}" style="border-width: 1px; border-color: #b20600; border-style: double;" alt="">

Good thing that I decided to include charts in the game or otherwise I would have missed this. And it didn't take much. All I needed to do is save the journey data and then display it. Moreover, Swift Charts allows you to show live charts which brings dynamisms to the game.

Every time we update the building occupancy, we save a log entry.

```swift
let newPeople = peopleArriving(hour: hour, day: day)
let peopleGoingOut = goingOut(hour: hour, day: day)
currentOccupancy += newPeople - peopleGoingOut
generateCalls(newPeople: newPeople, peopleGoingOut: peopleGoingOut)
log.append(
    LogEntry(
        date: date,
        occupancy: currentOccupancy,
        groundFloorQueue: hallCalls.filter { $0.from == 0 }.count
    )
)
```

The code to show the charts is pretty straightforward.
```swift
Button("Stats") {
    showingStats.toggle()
}
.sheet(isPresented: $showingStats) {
    GroupBox("Building Occupancy") {
        Chart {
            ForEach(dispatcher.building.log) { item in
                BarMark(x: .value("Minute", item.date, unit: .minute),
                        y: .value("People", item.occupancy))
            }
        }
    }
    .padding([.horizontal, .top])
    GroupBox("Ground Floor Queue") {
        Chart {
            ForEach(dispatcher.building.log) { item in
                BarMark(x: .value("Minute", item.date, unit: .minute),
                        y: .value("Ground Floor Q", item.groundFloorQueue))
            }
        }
    }
    .padding()
    .presentationDetents([.medium, .large])
}
```
After fixing the issue with the daily crowd generation, I realised I needed to wait until the game generates enough data to see the result. I decided to add a button to generate a day worth of data in a second. That way I could see the results immediately.

<img src="{{ '/assets/img/lift-stats-fixed.png' | prepend: site.baseurl }}" style="border-width: 1px; border-color: #b20600; border-style: double;" alt="">

