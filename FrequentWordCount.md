```dataviewjs
const stopWords = new Set([
    "the", "and", "a", "an", "to", "of", "in", "is", "it", "that",
    "for", "on", "as", "with", "was", "are", "this", "be", "by",
    "or", "from", "at", "but", "not", "have", "has", "had", "they",
    "you", "we", "he", "she", "their", "our", "your", "i", "my",
    "can", "will", "would", "could", "should", "do", "does", "did",
    "so", "if", "than", "then", "there", "what", "when", "which",
    "who", "how", "all", "more", "also", "about", "into", "up",
    "out", "no", "yes", "its", "these", "those", "word" , "words"
]);

const counts = {};

for (const page of dv.pages()) {
    const file = app.vault.getAbstractFileByPath(page.file.path);
    if (!file) continue;

    const content = await app.vault.read(file);

    const words = content
        .toLowerCase()
        .replace(/https?:\/\/\S+/g, "")
        .replace(/[^a-z0-9'-]+/g, " ")
        .split(/\s+/)
        .filter(word => word.length > 2)
        .filter(word => !stopWords.has(word));

    for (const word of words) {
        counts[word] = (counts[word] || 0) + 1;
    }
}

const results = Object.entries(counts)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 50);

dv.table(
    ["Rank", "Word", "Frequency"],
    results.map(([word, count], i) => [i + 1, word, count])
);
```