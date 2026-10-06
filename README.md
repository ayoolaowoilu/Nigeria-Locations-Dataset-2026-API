# 🇳🇬 Nigeria Locations Dataset

A structured JSON dataset of Nigerian **states**, **Local Government Areas (LGAs)** and **areas / neighbourhoods**, with a ready-to-use React location autocomplete component.

- 37 states (36 states + the Federal Capital Territory)
- 774 LGAs (including the 6 FCT area councils)
- Areas / towns / neighbourhoods nested under each LGA
- Works offline: import the JSON directly, no API calls or API keys

---

## Table of Contents

- [Data Format](#data-format)
- [Field Reference](#field-reference)
- [TypeScript Types](#typescript-types)
- [Quick Start](#quick-start)
- [Fetch via Raw GitHub URL](#fetch-via-raw-github-url)
- [Location Autocomplete Component](#location-autocomplete-component)
- [Project Structure](#project-structure)
- [Data Rules](#data-rules)
- [Contributing](#contributing)
- [License](#license)

---

## Data Format

The data lives in a single file: [`src/data/nigerialocc.json`](src/data/nigerialocc.json).

The hierarchy is **Country → State → LGA → Area**:

```
Nigeria
└── State        (e.g. Lagos)
    └── LGA      (e.g. Agege)
        └── Area (e.g. Oko Oba)
```

### Example

```json
{
  "country": "Nigeria",
  "totalStates": 37,
  "totalLgas": 774,
  "states": [
    {
      "name": "Lagos",
      "capital": "Ikeja",
      "lgas": [
        {
          "name": "Agege",
          "areas": ["Agege Town", "Oko Oba", "Orile Agege", "Dopemu"]
        }
      ]
    }
  ]
}
```

---

## Field Reference

### Root object

| Field         | Type       | Description                          |
| ------------- | ---------- | ------------------------------------ |
| `country`     | `string`   | Country name (`"Nigeria"`)           |
| `totalStates` | `number`   | Number of states, including the FCT  |
| `totalLgas`   | `number`   | Total number of LGAs nationwide      |
| `states`      | `State[]`  | List of all states                   |

### State

| Field     | Type     | Description                       |
| --------- | -------- | --------------------------------- |
| `name`    | `string` | State name, e.g. `"Lagos"`        |
| `capital` | `string` | State capital, e.g. `"Ikeja"`     |
| `lgas`    | `Lga[]`  | LGAs in the state                 |

### LGA

| Field   | Type       | Description                                          |
| ------- | ---------- | ---------------------------------------------------- |
| `name`  | `string`   | LGA name, e.g. `"Agege"`                             |
| `areas` | `string[]` | Towns, districts or neighbourhoods inside the LGA    |

---

## TypeScript Types

```ts
export interface NigeriaLocations {
  country: string;
  totalStates: number;
  totalLgas: number;
  states: State[];
}

export interface State {
  name: string;
  capital: string;
  lgas: Lga[];
}

export interface Lga {
  name: string;
  areas: string[];
}
```

---

## Quick Start

### 1. Import the data

```ts
import locations from '@/data/nigerialocc.json';

console.log(locations.totalStates); // 37
```

> If TypeScript can't resolve the import, set `"resolveJsonModule": true` in your `tsconfig.json`. Next.js enables this by default.

### 2. Common lookups

```ts
import locations from '@/data/nigerialocc.json';

// All state names
const stateNames = locations.states.map((s) => s.name);

// All LGAs in a state
const lagosLgas = locations.states
  .find((s) => s.name === 'Lagos')
  ?.lgas.map((l) => l.name);

// All areas in an LGA
const agegeAreas = locations.states
  .find((s) => s.name === 'Lagos')
  ?.lgas.find((l) => l.name === 'Agege')?.areas;
```

### 3. Flatten into a searchable list

```ts
const flat = locations.states.flatMap((state) =>
  state.lgas.flatMap((lga) => [
    { label: `${lga.name}, ${state.name}`, type: 'lga' },
    ...lga.areas.map((area) => ({
      label: `${area}, ${lga.name}, ${state.name}`,
      type: 'area',
    })),
  ]),
);
```

---

## Fetch via Raw GitHub URL

You don't have to clone the repo. The JSON can be loaded straight from GitHub's raw content URL:

```
https://raw.githubusercontent.com/<username>/<repo>/main/src/data/nigerialocc.json
```

Replace `<username>` and `<repo>` with your own, and adjust the path if you keep the file somewhere else (for example, at the repo root it is just `.../main/nigerialocc.json`).

### JavaScript / TypeScript

```ts
const URL =
  'https://raw.githubusercontent.com/<username>/<repo>/main/src/data/nigerialocc.json';

const res = await fetch(URL);
const locations: NigeriaLocations = await res.json();

console.log(locations.totalLgas); // 774
```

### cURL

```bash
curl -O https://raw.githubusercontent.com/<username>/<repo>/main/src/data/nigerialocc.json
```

> **Tip:** raw URLs are cached by GitHub for a few minutes and are not meant for heavy production traffic. For production apps, download the file and import it locally (see [Quick Start](#quick-start)).

---

## Location Autocomplete Component

`LocationAutocomplete` is a React / Next.js component (Tailwind CSS + `lucide-react`) that searches this dataset locally as you type.

```tsx
import LocationAutocomplete from '@/components/LocationAutocomplete';

export default function Page() {
  const [location, setLocation] = useState('');

  return <LocationAutocomplete value={location} onChange={setLocation} />;
}
```

### What users see

| Level | Main text | Secondary text         | Value returned to `onChange` |
| ----- | --------- | ---------------------- | ---------------------------- |
| State | Lagos     | State, Nigeria         | `Lagos`                      |
| LGA   | Agege     | LGA, Lagos State       | `Agege, Lagos`               |
| Area  | Oko Oba   | Agege LGA, Lagos State | `Oko Oba, Agege, Lagos`      |

### Features

- Instant, synchronous search with no network requests
- Multi-word queries (`lekki lagos`, `Oko Oba, Agege`)
- Case and accent insensitive
- Ranked results: exact match, then prefix match, then word match, then contains
- Keyboard navigation (↑ ↓ Enter Esc)
- Clear button and empty-state message

### Dependencies

```bash
npm install lucide-react
```

---

## Project Structure

```
.
├── src/
│   ├── components/
│   │   └── LocationAutocomplete.tsx
│   └── data/
│       └── nigerialocc.json
└── README.md
```

---

## Data Rules

To keep the dataset consistent, every entry should follow these rules:

1. Every state has `name`, `capital` and a `lgas` array.
2. Every LGA has `name` and an `areas` array (use `[]` if there are none yet).
3. `areas` is an array of plain strings, not objects.
4. Use proper capitalisation: `"Oko Oba"`, not `"oko oba"`.
5. No duplicate LGAs inside a state and no duplicate areas inside an LGA.
6. Keep `totalStates` and `totalLgas` in sync with the actual data.

---

## Contributing

Contributions are welcome, especially for adding or correcting **areas**.

1. Fork the repo
2. Create a branch: `git checkout -b add-areas-ikeja`
3. Edit `src/data/nigerialocc.json` following the [Data Rules](#data-rules)
4. Open a pull request describing what you added or fixed

If you spot a wrong name, spelling or missing LGA, please open an issue.

---

## License

Add your preferred license here (e.g. [MIT](https://opensource.org/licenses/MIT)).
