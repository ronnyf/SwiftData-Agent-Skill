---
name: swiftdata-pro
description: Writes, reviews, and improves SwiftData code using modern APIs and best practices. Use when reading, writing, or reviewing code that uses `@Model`, `@Query`, `#Predicate`, `FetchDescriptor`, `ModelContainer` / `ModelContext`, `@Relationship` (delete rules), `@Attribute(.unique)`, SwiftData + CloudKit, or model indexing / class inheritance.
license: MIT
metadata:
  author: Paul Hudson
  version: "1.0"
---

Write and review SwiftData code for correctness, modern API usage, project conventions. Report genuine problems only — no nitpicks, no invented issues.

**Review process** — step → reference → what it covers:

1. Core SwiftData issues → `references/core-rules.md` — autosaving, relationships, delete rules, property restrictions, FetchDescriptor optimization.
2. Predicates safe + supported → `references/predicates.md` — supported operations, patterns that crash at runtime, unsupported methods.
3. **Project uses CloudKit:** → `references/cloudkit.md` — uniqueness, optionality, eventual consistency.
4. **Targets iOS 18+:** indexing opportunities → `references/indexing.md` — single + compound property indexes.
5. **Targets iOS 26+:** class inheritance → `references/class-inheritance.md` — @available requirements, schema setup, predicate filtering.

Partial work: load only the relevant reference files.

## Core Instructions

- Target Swift 6.2+, modern Swift concurrency.
- SwiftData across the board (user preference). Suggest Core Data only for a feature SwiftData cannot solve.
- No third-party frameworks without asking first.
- Consistent project structure; folder layout by app feature.

## Output Format

Review request: findings organized by file. Per issue — (1) file + line(s), (2) rule violated, (3) brief before/after fix. Skip clean files. End with a prioritized summary: most impactful changes first.

Write/improve request: same rules, apply the changes directly instead of a findings report.

Example output:

### Destination.swift

**Line 8: Add an explicit delete rule for relationships.**

```swift
// Before
var sights: [Sight]

// After
@Relationship(deleteRule: .cascade, inverse: \Sight.destination) var sights: [Sight]
```

**Line 22: Never `isEmpty == false` in predicates – crashes at runtime. Use `!`.**

```swift
// Before
#Predicate<Destination> { $0.sights.isEmpty == false }

// After
#Predicate<Destination> { !$0.sights.isEmpty }
```

### DestinationListView.swift

**Line 5: `@Query` must only be used inside SwiftUI views.**

```swift
// Before
class DestinationStore {
    @Query var destinations: [Destination]
}

// After
class DestinationStore {
    var modelContext: ModelContext

    func fetchDestinations() throws -> [Destination] {
        try modelContext.fetch(FetchDescriptor<Destination>())
    }
}
```

### Summary

1. **Data loss (high):** missing delete rule, Destination.swift:8 – sights orphaned when a destination is deleted.
2. **Crash (high):** `isEmpty == false`, Destination.swift:22 – use `!isEmpty`.
3. **Incorrect behavior (high):** `@Query` on line 5 of DestinationListView.swift only works inside SwiftUI views.

End of example.
