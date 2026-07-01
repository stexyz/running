# Race type classification

Heuristics to assign a `type` value from name, distance, and notes.

## Allowed values

`road-marathon`, `road-half`, `road-10k`, `road-5k`, `trail`, `ultra`, `vertical`, `duathlon`, `night-race`, `other`

## Priority order for classification

Evaluate top to bottom. First match wins.

### 1. Ultra

- Distance ≥ 43 km, OR
- Name contains: `ultra`, `100`, `50k`, `50 km`

→ `ultra`

### 2. Vertical

- Name contains: `vertical`, `vk`, `výběh`, `do vrchu`

→ `vertical`

### 3. Duathlon

- Name contains: `duatlon`, `duathlon`

→ `duathlon`

### 4. Night race

- Name contains: `noční`, `night`, `nocturne`

→ `night-race`

(Overrides road distance classifications — a "noční půlmaraton" is `night-race`, not `road-half`.)

### 5. Trail

- Notes indicate trail surface, OR
- Name contains: `trail`, `trailový`, `horský`, `krosový`, `terénní`, `lesní`

→ `trail`

### 6. Road marathon

- Distance between 41.5 and 43 km, AND not trail/ultra/night above

→ `road-marathon`

### 7. Road half-marathon

- Distance between 20.5 and 21.5 km, AND not trail/ultra/night above

→ `road-half`

### 8. Road 10k

- Distance between 9.5 and 10.5 km, AND not trail/night above

→ `road-10k`

### 9. Road 5k

- Distance between 4.5 and 5.5 km, AND not trail/night above

→ `road-5k`

### 10. Fallback

- Anything else → `other`

## Notes

- If `distance_km` is `null`, only name-based rules can fire; anything unclassified → `other`.
- The classification is heuristic. Users can correct race types in plan files manually.
