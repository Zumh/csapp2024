## Ahmdal Law
- Speedup = 1 / [(1 - P) + (P / N)]
### Practice Problem 1.1
- Suppose you work as a truck driver, and you have been hired to carry a load of potatoes from Boise, Idaho, to Minneapolis, Minnesota, a total distance of 2,500 kilometers.
- You estimate you can average 100 km/hr driving within the speed limits, requiring a total of 25 hours for the trip.

#### Aside

- Expressing relative performance old old Tnew=(1−α)Told+(αTold)/k=Told[(1−α)+α/k] old new S=1(1−α)+α/k (1.1)

- The best way to express a performance improvement is as a ratio of the form T /T , where T is the time required for the original version and T is the time required by the modified version.
- This will be a number greater than 1.0 if any real improvement occurred. We use the suffix '×' to indicate such a ratio, where the factor "2.2×" is expressed verbally as "2.2 times."
- The more traditional way of expressing relative change as a percentage works well when the change is small, but its definition is ambiguous.
- Should it be 100 · (T − T )/T , or possibly 100 · (T − T )/T , or something else? In addition, it is less instructive for large changes.
- Saying that "performance improved by 120%" is more difficult to comprehend than simply saying that the performance improved by 2.2×.

A. You hear on the news that Montana has just abolished its speed limit, which constitutes 1,500 km of the trip. Your truck can travel at 150 km/hr. What will be your speedup for the trip?

B. You can buy a new turbocharger for your truck at [www.fasttrucks.com](http://www.fasttrucks.com). They stock a variety of models, but the faster you want to go, the more it will cost. How fast must you travel through Montana to get an overall speedup for your trip of 1.67×?

### Practice Problem 1.2

- The marketing department at your company has promised your customers that the next software release will show a 2× performance improvement.
- You have been assigned the task of delivering on that promise. You have determined that only 80% of the system can be improved.
- How much (i.e., what value of k) would you need to improve this part to meet the overall performance target?

#### Explain
- One interesting special case of Amdahl's law is to consider the effect of setting k to ∞.
- That is, we are able to take some part of the system and speed it up to the point at which it takes a negligible amount of time. We then get So, for example, if we can speed up 60% of the system to the point where it requires close to no time, our net speedup will still only be 1/0.4 = 2.5×.

- Amdahl's law describes a general principle for improving any process.
- In addition to its application to speeding up computer systems, it can guide a company trying to reduce the cost of manufacturing razor blades, or a student trying to improve his or her grade point average. - Perhaps it is most meaningful in the world of computers, where we routinely improve performance by factors of 2 or more. Such high factors can only be achieved by optimizing large parts of a system.

