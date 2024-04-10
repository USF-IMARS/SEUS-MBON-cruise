## Data Pipeline Overview
```mermaid
graph LR

CRUISE((MBON Cruise))

%% === cruise outputs:
%% === cruise outputs:
%% physical samples
CRUISE --> HPLC>HPLC Pads]
CRUISE --> ZOO>Zooplankton Sample \n in Collection]
CRUISE --> PADS>Absorption \n filter pads]
CRUISE --> CDOM>CDOM Samples]
CRUISE --> eDNA>eDNA samples]
%% data files
CRUISE --> BB3[[BB3 Datafile]]
CRUISE --> RAD[[Radiance Datafile]]
CRUISE --> CTD_DEPTH[[CTD Data from TAMU ERDDAP]]
CRUISE --> LOGSHEET[["
  FKNMS_sample_logsheet_{m}_{y}.xlsx
  logsheet.
  (water filtration metadata,
  Zooplankton sampling metadata,
  eDNA sampling metadata,
  BB3 sampling metadata)
"]]

%% === inventory pipeline
LOGSHEET --> ROWS_GEN{status_inventory_gen}
  --> INVENTORY[(Cruise Status+Inventory \n Spreadsheet)]

%% === eDNA pipeline
eDNA[[eDNA samples]]
--> NOAA{logsheet_mod}
--> NOAA_LOGSHEET
LOGSHEET --> NOAA

%% === BB3 Pipeline
LOGSHEET --> BB3_QC
BB3 --> BB3_QC{"bb3_qc"}
--> BB3_FIXED["BB3 Datafile (fixed)"]
--> MERGE_BB3{bb3_depth_merge} 
--> BB3_N_DEPTH["BB3+Depth File"]
CTD_DEPTH --> MERGE_BB3

BB3_N_DEPTH--> BB3_2_BS{"bb3_backscatter_conv"}
--> BS_F
--> SB_FMT

%% === reflectance pipeline
RAD -->
RAD_CONV{"rad_2_rrs"}
--> RRS_F["RRS File"]
--> SB_FMT

%% === HPLC pipeline
LOGSHEET --> NASA
HPLC --> NASA{hplc_proc}
NASA --> HPLC_F[HPLC File]
HPLC_F --> SB_FMT
  --> SEABASS

%% === zooplankton pipeline
ZOO --> TAXIZE{taxonomist \n microscopy}
TAXIZE --> ZOO_LOG[["zooplankton log spreadsheets \n (up to 11/cruise)"]]
LOGSHEET --> ZOO_MERGE{Zooplankton \n files merge}
ZOO_LOG --> ZOO_MERGE
ZOO_MERGE --> WORMS_ALIGN{WoRMS \n alignment}
WORMS_ALIGN --> ZOO_COMBINED[[combined \n zooplankton file]]
ZOO_COMBINED --> DWC_CREATE{DwC Creation}
DWC_CREATE --> DWC[[DwC Archive]] --> OBIS[(OBIS)]
DWC_CREATE --> extra
ZOO --> STORE[("IMaRS Storage Room(s)")]

%% === Absorption filter pad + CDOM pipeline
PADS --> PAD_EXTRACT{abs_pad_exr}
PAD_EXTRACT --> AP
PAD_EXTRACT --> AD
PAD_EXTRACT --> CHLOR

LOGSHEET --> PAD_P
AP --> PAD_P{"abs_qc"}
AD --> PAD_P
CHLOR -->PAD_P

CDOM --> CDOM_MEASURE{cdom_measure}
CDOM_MEASURE --> CDOM_F[CDOM File]
--> PAD_P

PAD_P --> aP --> SB_FMT{sb_fmt}
PAD_P --> aPH --> SB_FMT
PAD_P --> aD --> SB_FMT

PAD_P --> aCDOM --> SB_FMT
CHLOR --> SB_FMT


%% things that need to go to SEABASS
%%SPECTRA[[Absorption Pad Spectra]] --> SEABASS
%%CDOM --> SEABASS
%%HPLC --> SEABASS
%%REFLECTANCE --> SEABASS

%%subgraph legend
%%    SAMPLE{Physical Object}
%%    DB[(Database)]
%%    UI[[Interface]]
%%end 

%% this "clickable" class works magically? 
classDef clickable color:#25f,text-decoration: underline

%% custom classes
classDef popupsubgraph color:#2ff,text-decoration: underline

%% sub-graphs from within this repo
%%click DWC_ALIGN "https://ioos.github.io/mbon-docs/mbon-data-flow.html#darwin-core-alignment"
%%DWC_ALIGN:::popupsubgraph

%% external links
click MAP "https://github.com/marinebon/map-of-activities"
click REGISTER "https://github.com/marinebon/dataset-registry/issues"

```
### Pipeline processes:
process              | description
---------------------|-------------------------------------------
status_inventory_gen | usf-imars/mbon_cruise_scripts::merge_meta_chl_hplc_cdom.Rmd QC on logsheets & create collection Event Rows (station_id + stations) in the status inventory sheet.
bb3_qc               | Sebastian has a WiP QC python script
bb3_depth_merge      | Merge BB3 & Depth data - TODO: how?
bb3_backscatter_conv | convert BB3+Depth data into backscatter - TODO: how?
rad_2_rrs            | Radiance to RRS conversion - TODO: how?
hplc_proc            | HPLC Processing done by Crystal @ NASA
taxon_id             | Taxonomist IDs Plankton from sample aliquot(s)
dwc_align            | DwC Alignment
abs_pad_extr         | extraction from filter pads
abs_qc               | Aborbance post-processing done by Jenn
cdom_measure         | CDOM Measurement taken
sb_fmt               | format and upload data to SEABASS
logsheet_mod         | modify logsheets & send to NOAA (Luke Thompson + Enrique)
  


