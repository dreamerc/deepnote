802.11 Design Datasheets/Suggestions

# Standards
| Name     | Freq(Hz)    | Bandwidth(Hz)        | Modulator                | MaxSpeed(bit/s)                                           |
| -------- | ----------- | -------------------- | ------------------------ | --------------------------------------------------------- |
| 802.11a  | 5 G         | 5/10/20 M            | OFDM                     | 13.5/27/54 M                                              |
| 802.11ac | 2.4 G       | 20/40/80/80+80/160 M | MCS 7/256-QAM(/1024-QAM) | 150/300/400/450/600/(750/)800(/1000) M                    |
|          | 5 G         | 20/40/80/80+80/160 M | MCS 7/MCS 9/1024-QAM     | 433/650/867/975/1300/1625/1733/2167/1300+1300/2167+2167 M |
| 802.11ad | 60 G        | 2/160 M              | OFDM                     | 0.85/6.7 G                                                |
| 802.11ax | 2.4/5(/6) G | 20/40/80/80+80/160 M | MIMO-OFDM/OFDMA          | 1147/2294/4804/9608 M                                     |
| 802.11b  | 2.4 G       | 22 M                 | DSSS                     | 11 M                                                      |
| 802.11g  | 2.4 G       | 20 M                 | OFDM                     | 54 M                                                      |
| 802.11n  | 2.4 G       | 20/40 M              | MIMO-OFDM                | 288.8/600 M                                               |

# Non-overlapping Channels
| Freq(Hz) | Bandwidth(Hz) | Channels                                                                                    |
| -------- | ------------- | ------------------------------------------------------------------------------------------- |
| 2.4 G    | 20 M          | 1,6,11,14                                                                                   |
|          | 40 M          | 1,6,11                                                                                      |
|          | 80 M          | 3,9                                                                                         |
|          | 80+80 M       | 3,9                                                                                         |
|          | 160 M         | 5                                                                                           |
| 5 G      | 20 M          | 36,40,44,48,52,56,60,64,100,104,108,112,116,120,124,128,132,136,140,144,149,153,157,161,165 |
|          | 40 M          | 36,44,52,60,100,108,116,124,132,140,149,157,167                                             |
|          | 80 M          | 40,52,64,104,116,128,144,149,161                                                            |

# Tools
| Name                       | Type     | Free/Pay | Link                                                  |
| -------------------------- | -------- | -------- | ----------------------------------------------------- |
| TamoGraph Site Survey      | Design   | Pay      |                                                       |
| Ekahau HeatMapper          | Design   | Free/Pay |                                                       |
| SolarWinds Wifi Heat Map   | Design   | Pay      |                                                       |
| NetSpot                    | Design   | Pay      |                                                       |
| VisiWave                   | Design   | Pay      |                                                       |
| AirMagnet Survey PRO       | Design   | Pay      |                                                       |
|                            |          |          |                                                       |
| Python-wifi-survey-heatmap | Analysis | Free     | https://github.com/jantman/python-wifi-survey-heatmap |