# DIY Child GPS Tracker

A small, standalone GPS tracker that reports its current or last known coordinates to a self-hosted Traccar server on Unraid. Nextcloud integration can be considered later.

> **Project status:** Prototype plan. Component prices and availability must be reconfirmed before purchase. The development board is for testing, not the final keyring-sized enclosure.

## 1. Goals

- Standalone device with no phone required.
- GNSS location and cellular data transmission.
- Report location every 5 minutes.
- Send coordinates to Traccar hosted on Unraid.
- Retain the last valid GPS fix when GPS or cellular reception is unavailable.
- Minimum target of 24 hours' battery life; 48 hours is a stretch goal.
- Minimise final size, aiming for keyring-sized hardware.
- Keep location data on infrastructure under the owner's control.

## 2. Proposed architecture

```text
GNSS satellites
      |
      v
BG95-M3 cellular/GNSS module
      |
      | LTE-M data
      v
Traccar OsmAnd protocol endpoint
      |
      v
Traccar map, history and last-update time
      |
      +---- Possible Nextcloud integration later
```

The prototype will use a Waveshare BG95-M3 Zero development board. The eventual compact device should use a custom PCB around a cellular/GNSS module, power-management circuitry, antennas and a rechargeable battery.

## 3. Initial component selection

### Main board

**Waveshare BG95-M3 Zero development board**

- Combines cellular connectivity, GNSS and a programmable controller.
- Supports LTE-M/NB-IoT; exact capabilities depend on the module variant and network.
- Intended for prototyping and firmware development.
- Indicative listing previously found at Opencircuit: €62.40 excluding VAT and shipping.
- Supplier: <https://opencircuit.se/product/bg95-evb-development-board-designed-quecpython>

Before ordering, verify the exact board revision, included antennas, battery input/charging circuitry, SIM format, module frequency bands and firmware support.

### Bill of materials

| Item | Specification / notes | Estimated cost |
|---|---|---:|
| Cellular/GNSS development board | Waveshare BG95-M3 Zero | SEK 900–1,100 including estimated VAT/delivery |
| Rechargeable battery | 3.7 V LiPo, approximately 1,000–1,500 mAh; verify dimensions, connector polarity and protection | SEK 80–180 |
| LTE/GNSS antennas | Use included antennas if suitable; otherwise match frequency bands and connectors | SEK 0–250 |
| SIM and mobile service | SIM and plan explicitly supporting LTE-M in Sweden; confirm coverage, roaming and data terms | SEK 50–150/month budget estimate |
| Plastic enclosure | Small non-metallic case for prototype; final case to be custom-designed | SEK 40–120 |
| Wiring and connectors | Suitable cable, connectors and basic assembly materials | SEK 30–80 |
| Debugging accessories | USB cable, jumper wires and USB-UART adapter if required | SEK 100–200 |

**Estimated prototype hardware total:** approximately SEK 1,250–1,900, plus mobile service. This is a planning estimate, not a verified checkout basket. Prices, VAT, delivery, included accessories and stock must be checked with suppliers before purchase.

### Suppliers to check

- Opencircuit Sweden: <https://opencircuit.se/>
- Electrokit: <https://www.electrokit.com/>
- DigiKey Sweden: <https://www.digikey.se/>
- Mouser Sweden: <https://www.mouser.se/>
- TME: <https://www.tme.eu/>
- Telia IoT: <https://www.telia.se/en/business/iot/managed-connectivity>
- Telenor IoT: <https://iot.telenor.com/technologies/connectivity/lte-m/>

The cellular plan and module must be compatible. Do not assume a normal mobile phone SIM supports LTE-M.

## 4. Server integration: Traccar on Unraid

Use Traccar's **OsmAnd protocol** for the first prototype. It accepts location updates over HTTP using fields such as a device identifier, latitude, longitude and timestamp.

1. Create a device in Traccar with a unique identifier, for example `CHILDTRACK01`.
2. Confirm the OsmAnd protocol is enabled and note the configured port (commonly `5055`).
3. Confirm network routing/firewall rules allow the tracker to reach Traccar.
4. Configure the tracker to send a request containing the device ID, latitude, longitude and UTC timestamp.
5. Confirm that updates appear in the Traccar map and history.

Documentation: <https://www.traccar.org/osmand/>

### Security

- Do not expose an unauthenticated location endpoint to the public internet.
- Use TLS for internet-facing traffic and restrict access with suitable authentication or a private VPN.
- Use a unique device identifier and protect any credentials.
- Limit who can view location history.
- Treat this as a prototype, not a dependable child-safety device, until reliability and security testing are complete.

## 5. Firmware behaviour

Expected reporting loop:

1. Wake from low-power sleep.
2. Obtain a GNSS fix, with a defined timeout.
3. If the fix is valid, save coordinates, accuracy and the timestamp of the fix.
4. If no fix is available, retain the previous valid coordinates.
5. Attempt to transmit the latest location record.
6. If transmission fails, queue the record or schedule a retry.
7. Sleep until the next reporting cycle.

Important: when sending an old location, preserve the timestamp of the original GPS fix. Never make an old position appear current merely because it was retransmitted. The server/UI should clearly show the last fix time and last successful transmission time.

Suggested fields:

- Device ID
- Latitude and longitude
- GPS-fix timestamp in UTC
- Accuracy, if available
- Battery voltage or estimated battery level
- Optional cellular signal information
- Status indicating whether the location is a fresh fix or last known position

## 6. Battery-life assumptions

A 1,500 mAh battery has the following idealised runtime before conversion losses, battery reserve, ageing and other real-world effects:

| Average current | Idealised runtime |
|---:|---:|
| 10 mA | 150 hours |
| 25 mA | 60 hours |
| 50 mA | 30 hours |
| 75 mA | 20 hours |

These are calculations, not measured performance predictions. GNSS acquisition time, weak cellular signal, modem registration, retransmissions, power-management design and sleep current can materially change runtime.

Initial targets:

- Reporting interval: 5 minutes.
- Minimum measured endurance: 24 hours.
- Stretch target: 48 hours.
- Battery: start around 1,500 mAh for testing, then reduce battery size if measurements support it.

Measure current consumption and run a real-world test before choosing the final battery and enclosure. Verify the development board's charging circuit and battery requirements before connecting a cell. Do not assume a generic TP4056 charger module is appropriate.

## 7. Staged build plan

### Stage 1 — Validate hardware

- [ ] Order one BG95-M3 Zero board and one compatible battery.
- [ ] Check the exact board documentation and supplied antennas.
- [ ] Confirm the modem registers on a supported Swedish LTE-M network.
- [ ] Obtain a GNSS fix outdoors.
- [ ] Record board revision, firmware version and module variant.

### Stage 2 — Send locations to Traccar

- [ ] Create a Traccar device ID.
- [ ] Confirm OsmAnd protocol configuration and connectivity.
- [ ] Implement the 5-minute reporting loop.
- [ ] Send coordinates and the original fix timestamp.
- [ ] Verify map display, location history and timestamps.
- [ ] Test behaviour when GPS or mobile data is unavailable.

### Stage 3 — Optimise power and reliability

- [ ] Implement sleep and wake behaviour.
- [ ] Add GPS timeout and cellular retry logic.
- [ ] Preserve the last valid fix during outages.
- [ ] Track battery voltage.
- [ ] Measure current consumption in each operating state.
- [ ] Run a minimum 48-hour test under realistic conditions.
- [ ] Test indoors, outdoors, in a pocket/bag and under weak signal.

### Stage 4 — Miniaturise

- [ ] Select the bare cellular/GNSS module for a custom PCB.
- [ ] Design the power supply and charging circuit to the module's specifications.
- [ ] Select suitable GNSS and cellular antennas.
- [ ] Design a compact plastic enclosure.
- [ ] Check antenna performance in the assembled enclosure.
- [ ] Re-test battery endurance and cellular reliability.

### Stage 5 — Controlled real-world testing

- [ ] Add low-battery reporting.
- [ ] Add stale-location indication and missed-update alerts on the server.
- [ ] Secure the endpoint and restrict location access.
- [ ] Test charging, drop resistance and temperature within component specifications.
- [ ] Document limitations and failure modes before relying on it for safety.

## 8. Size and design constraints

The development board will be much larger than the final target. Keyring-sized construction requires a custom PCB and careful antenna and battery selection. The battery, GNSS antenna and cellular antenna all need physical space. A smaller case can reduce reception performance, so final size should be determined after testing.

## 9. Open questions and risks

- Confirm the exact BG95-M3 Zero revision and its power/charging implementation.
- Confirm current supplier pricing, stock, shipping and VAT.
- Confirm LTE-M coverage, supported bands and SIM-plan terms in the intended areas of Sweden.
- Establish measured average and peak current before selecting the final battery.
- Decide how the tracker will authenticate to Traccar securely.
- Decide whether Nextcloud integration is needed after the Traccar prototype works.

## 10. References

- Traccar OsmAnd protocol: <https://www.traccar.org/osmand/>
- Traccar supported devices: <https://www.traccar.org/devices/>
- Opencircuit: <https://opencircuit.se/>
- Electrokit: <https://www.electrokit.com/>
- DigiKey Sweden: <https://www.digikey.se/>
- Mouser Sweden: <https://www.mouser.se/>
- TME: <https://www.tme.eu/>
- Telia IoT: <https://www.telia.se/en/business/iot/managed-connectivity>
- Telenor IoT: <https://iot.telenor.com/technologies/connectivity/lte-m/>

---

**Reminder:** This is a prototype plan, not a certified or proven child-safety product. Make location sharing transparent, restrict access to location data, and do not rely on the tracker as the sole safety measure.
