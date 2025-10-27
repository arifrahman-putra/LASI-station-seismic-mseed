# LASI Seismic Station Data

This repository contains seismic data in MSEED format from the LASI observation station, analyzed in the paper:

**Inverse Parameter Estimation of Seismic Sensing Systems Using SIREN-based Physics-Informed Neural Networks: A Seismometer Case Study**  
Authors: Arifrahman Yustika Putra, Adhi Harmoko Saputro, Titik Lestari  
Journal: Earth Science Informatics (submitted)  
DOI: [will be added upon publication]

## Data Description

- **Station**: LASI
- **Source**: Indonesian Agency for Meteorological, Climatological, and Geophysics (BMKG)
- **Format**: MSEED (Mini-SEED)
- **Station Metadata**: StationXML format (inventory)

## File Structure

The repository is organized into two main folders:

### 1. Mseed Folder
Seismic waveform data organized hierarchically by year, network, station, and channel:
```
Mseed/
└── YYYY/                    # Year (e.g., 2020, 2021, 2022)
    └── NETWORK/             # Network code (e.g., IA)
        └── STATION/         # Station code (e.g., LASI)
            └── CHANNEL/     # Channel code (e.g., SHZ.D, SHN.D, SHE.D)
                └── *.mseed  # MSEED files
```

**Example structure:**
```
Mseed/
└── 2020/
    └── IA/
        └── LASI/
            ├── SHZ.D/
            │   ├── IA.LASI..SHZ.D.2020.002
            │   ├── IA.LASI..SHZ.D.2020.003
            │   └── ...
            ├── SHN.D/
            │   └── ...
            └── SHE.D/
                └── ...
```

**Naming Convention:**  
Files follow the format: `NETWORK.STATION..CHANNEL.YEAR.DAYOFYEAR`

### 2. Inventory Folder
Station metadata in StationXML format containing instrument response information:
```
Inventory/
└── LASI.xml              # Station inventory file
```

The inventory file contains:
- Sensor and digitizer types
- Sampling rates
- Instrument response information
- Data availability periods

## Usage

These MSEED files can be read using standard seismological software. Below is an example using **ObsPy (Python)**:
```python
from obspy import UTCDateTime
from obspy.core.stream import Stream
from obspy import read_inventory
from obspy.clients.filesystem.sds import Client
import numpy as np

StationID = "LASI"  # station code
Channel = "SHZ"
StartTime = UTCDateTime("2023-1-2T12:00:00.000000Z")
EndTime = StartTime + 3600

freq_min = 1 / 500  # lower frequency limit (upper period = 500s)
freq_max = 1 / 4    # upper frequency limit (lower period = 4s)

# Load station inventory
link_inv = "Inventory/" + StationID + ".xml"
inv = read_inventory(link_inv)

# Load waveform data
client = Client("Mseed")  # path to Mseed folder
st = client.get_waveforms("IA", StationID, "*", Channel, StartTime, EndTime, attach_response=True)
st.merge()  # merge all traces in st
tr = st[0]  # define tr as the trace content

# Masked array filling
if isinstance(tr.data, np.ma.MaskedArray):
    tr.data = tr.data.filled(0)
    print("Filled masked values with zeros")
else:
    print("No masked values found - data is already valid")

st = Stream(traces=[tr])

# Remove response with pre-filtering
filt_freqs = (0.5 * freq_min, freq_min, freq_max, 2 * freq_max)
st.remove_response(inventory=inv, output="DISP", pre_filt=filt_freqs)
st.resample(1.0)  # uses sampling rate of 1 hz
```

## Citation

If you use this data in your research, please cite:
```
Putra, A.Y., Saputro, A.H., Lestari, T. (2025). Inverse Parameter Estimation of 
Seismic Sensing Systems Using SIREN-based Physics-Informed Neural Networks: 
A Seismometer Case Study. Earth Science Informatics. [DOI when available]
```

## Acknowledgements

The seismic data was provided by the Indonesian Agency for Meteorological, Climatological, and Geophysics (BMKG).

## License

The data is made available under MIT license.

## Contact

For questions regarding this dataset, please contact:  
1. First author (Arifrahman Yustika Putra): arifrahman.yustika31@ui.ac.id or arifrahman2703@gmail.com
2. Second and corresponding author (Adhi Harmoko Saputro): adhi@sci.ui.ac.id
