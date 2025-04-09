# Will Rogers Daily Telegram Locations

This repository contains a compressed version of Will Rogers' telegram location data (1926-1935).

## Rebuilding the CSV

To rebuild the full `dt.csv` from the compressed data:

```python
import csv
from datetime import datetime, timedelta

def compressed_to_csv(input_path, output_path):
    # Read compressed data
    with open(input_path, 'r', encoding='utf-8') as f:
        # Read header
        n_locations, n_sequences = map(int, f.readline().strip().split(','))
        
        # Read locations
        locations = []
        for _ in range(n_locations):
            name, lat, lon = f.readline().strip().split('|')
            locations.append([name, lat, lon])
            
        # Read sequences
        sequences = []
        for _ in range(n_sequences):
            start_date, end_date, loc_id, start_id, end_id = f.readline().strip().split('|')
            sequences.append([
                start_date,
                end_date, 
                int(loc_id),
                int(start_id),
                int(end_id)
            ])

    # Write CSV
    with open(output_path, 'w', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow(['id', 'byline_date', 'byline_place', 'lat', 'lon'])
        
        for seq in sequences:
            start = datetime.strptime(seq[0], '%Y-%m-%d')
            end = datetime.strptime(seq[1], '%Y-%m-%d')
            loc = locations[seq[2]]
            
            current_date = start
            current_id = seq[3]
            
            while current_date <= end:
                writer.writerow([
                    f'dt{current_id}',
                    current_date.strftime('%Y-%m-%d'),
                    loc[0],
                    loc[1],
                    loc[2]
                ])
                current_date += timedelta(days=1)
                current_id += 1

# Usage:
compressed_to_csv('dt.yaml', 'dt.csv')
```

The output CSV will have the following columns:

- `id`: Telegram ID (e.g. "dt1", "dt2")
- `byline_date`: Date in YYYY-MM-DD format
- `byline_place`: Location name
- `lat`: Latitude coordinate
- `lon`: Longitude coordinate

## Data Validation

To verify the reconstruction was successful:

1. The records should be in chronological order
2. IDs should be sequential
3. Each date should have exactly one location
4. The date range should be complete from 1926-1935
5. Expected row count: 2817 records
