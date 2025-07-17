// Helper function to make weighted random choice
function weightedChoice(weights) {
    const total = Object.values(weights).reduce((sum, weight) => sum + weight, 0);
    let random = Math.random() * total;

    for (const [choice, weight] of Object.entries(weights)) {
        random -= weight;
        if (random <= 0) {
            return choice;
        }
    }
    return Object.keys(weights)[0]; // Fallback
}

// Reversed prediction function (opposite logic)
function predictBigOrSmall(periodNumber) {
    const digits = periodNumber.toString().split('').map(Number);
    const digitSum = digits.reduce((sum, digit) => sum + digit, 0);

    // LEVEL 1: Inverted weight based on sum
    let level1Weights;
    if (digitSum >= 18) {  // High sum → originally favors "Big" → now favor "Small"
        level1Weights = { "Big": 0.3, "Small": 0.7 };
    } else if (digitSum >= 9) {  // Balanced → leave same
        level1Weights = { "Big": 0.5, "Small": 0.5 };
    } else {  // Low sum → originally favors "Small" → now favor "Big"
        level1Weights = { "Big": 0.7, "Small": 0.3 };
    }

    const level1Choice = weightedChoice(level1Weights);

    // LEVEL 2: Reverse influence of last digit
    const lastDigit = digits[digits.length - 1];
    let level2Weights;
    if (lastDigit % 2 === 0) {  // Even → originally favors "Big" → now favor "Small"
        level2Weights = level1Choice === "Big" ?
            { "Big": 0.4, "Small": 0.6 } :
            { "Big": 0.6, "Small": 0.4 };
    } else {  // Odd → originally favors "Small" → now favor "Big"
        level2Weights = level1Choice === "Big" ?
            { "Big": 0.6, "Small": 0.4 } :
            { "Big": 0.4, "Small": 0.6 };
    }

    const level2Choice = weightedChoice(level2Weights);

    // LEVEL 3: Reverse middle digit influence
    const middleDigit = digits[1] || 0; // Safe fallback
    let level3Weights;
    if ([1, 3, 7].includes(middleDigit)) {
        // Originally favors Big → now favor Small
        level3Weights = level2Choice === "Big" ?
            { "Big": 0.5, "Small": 0.5 } :
            { "Big": 0.6, "Small": 0.4 };
    } else {
        // Originally favors Small → now favor Big
        level3Weights = level2Choice === "Big" ?
            { "Big": 0.6, "Small": 0.4 } :
            { "Big": 0.5, "Small": 0.5 };
    }

    const finalChoice = weightedChoice(level3Weights);
    return finalChoice;
}
