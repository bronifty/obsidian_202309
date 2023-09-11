
- triplets
```ts
function compareTriplets(a: number[], b: number[]): number[] {
    let score: number[] = [0, 0]; // Initialize scores: [Ana's score, Bob's score]

    for(let i = 0; i < 3; i++) {
        if(a[i] > b[i]) {
            score[0] += 1; // Increment Ana's score if her number is larger
        } else if(a[i] < b[i]) {
            score[1] += 1; // Increment Bob's score if his number is larger
        }
    }

    return score;
}

```

