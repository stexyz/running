# Sample race JSON

Canonical shape for a scraped race-list file. Use as a reference when writing output.

```json
{
  "source": "ceskybeh.cz",
  "fetched_at": "2026-07-01T10:23:00Z",
  "races": [
    {
      "id": "ceskybeh-2026-04-05-prazsky-pulmaraton-21k",
      "name": "Pražský půlmaraton",
      "date": "2026-04-05",
      "location": {"city": "Praha", "region": "Praha", "country": "CZ"},
      "distance_km": 21.0975,
      "type": "road-half",
      "url": "https://ceskybeh.cz/zavod/prazsky-pulmaraton",
      "source": "ceskybeh.cz"
    },
    {
      "id": "ceskybeh-2026-05-17-krosovy-behokolo-brd-10k",
      "name": "Krosový běh okolo Brd",
      "date": "2026-05-17",
      "location": {"city": "Příbram", "region": "Středočeský", "country": "CZ"},
      "distance_km": 10.0,
      "type": "trail",
      "url": "https://ceskybeh.cz/zavod/krosovy-beh-brd",
      "source": "ceskybeh.cz"
    },
    {
      "id": "ceskybeh-2026-06-21-nocni-desitka-brno-10k",
      "name": "Noční desítka Brno",
      "date": "2026-06-21",
      "location": {"city": "Brno", "region": "Jihomoravský", "country": "CZ"},
      "distance_km": 10.0,
      "type": "night-race",
      "url": "https://ceskybeh.cz/zavod/nocni-desitka-brno",
      "source": "ceskybeh.cz"
    }
  ]
}
```

## Notes on the examples

- Third entry demonstrates `night-race` overriding what would otherwise be `road-10k`.
- Second entry demonstrates trail classification from name keyword.
- `id` includes distance suffix when the event has multiple distances (see first entry — `-21k`).
