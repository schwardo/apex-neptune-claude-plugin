# Apex Neptune Claude Plugin

A Claude Code plugin for monitoring and analyzing data from your Apex Neptune aquarium controller. This plugin enables Claude to download sensor readings, track water parameters, and analyze historical aquarium trends through natural conversation.

## Features

- **Real-time Status Monitoring**: Check current tank parameters (temperature, pH, salinity, ORP)
- **Historical Data Analysis**: Retrieve and analyze sensor data over time
- **Equipment Tracking**: Monitor outlet states and equipment runtime
- **Intelligent Subagent**: Automatically invoked when you ask about your aquarium
- **Example Data**: Includes documented XML examples for understanding data formats

## Installation

### Prerequisites

- [Claude Code](https://claude.com/code) installed and configured
- Apex Neptune aquarium controller on your local network
- Controller accessible at `http://apex.local` (or configure your hostname)

### Install the Plugin

Install the Claude marketplace via Github:

```claude
/plugin marketplace add schwardo/apex-neptune-claude-plugin
```

Install the `apex-neptune-claude-plugin` plugin from the marketplace:

```claude
/plugin install apex-neptune-claude-plugin@apex-neptune-claude-plugin-repo
```

### Verify Installation

Start a new Claude Code session and verify the plugin is loaded:

```bash
claude
```

The `apex-neptune-retriever` subagent should be available automatically when you ask aquarium-related questions.

## Usage

Once installed, simply ask Claude about your aquarium. The plugin will automatically activate when needed.

### Examples

#### Check Current Tank Status

```
What's the current status of my reef tank?
```

Claude will fetch `status.xml` and show you:
- Current water temperature
- pH levels
- Salinity readings
- ORP values
- Outlet states (equipment on/off)
- Any alarms or issues

#### View Historical Temperature Data

```
Show me the temperature trends over the past week
```

Claude will:
- Fetch historical data from `datalog.xml`
- Analyze temperature patterns
- Identify any significant fluctuations
- Alert you to potential issues

#### Check Equipment Status

```
Is my heater currently running?
```

Claude will check the outlet states and tell you:
- Current heater state (ON/OFF/AON/AOF)
- Whether it's manually controlled or auto-controlled
- Recent activity from the outlet log

#### Analyze pH Trends

```
Has my pH been stable this month?
```

Claude will:
- Retrieve historical pH data
- Calculate statistics (min, max, average, standard deviation)
- Identify trends or drift
- Suggest actions if needed

#### Review Feeding Cycles

```
Show me when the automatic feeder ran today
```

Claude will check `outlog.xml` for feeder activity and related equipment states (pumps/skimmers that turn off during feeding).

#### Equipment Runtime Analysis

```
How often did the heater cycle on yesterday?
```

Claude will analyze `outlog.xml` to show:
- Number of on/off cycles
- Total runtime
- Potential issues (excessive cycling, no activity, etc.)

### Advanced Usage

#### Specify Date Ranges

```
What was the average temperature from January 20-27?
```

The plugin supports flexible date range queries and will fetch data in 7-day increments automatically.

#### Compare Parameters

```
Did my pH drop when the calcium reactor was offline last week?
```

Claude can correlate equipment states with parameter changes.

#### Program Review

```
Show me the programming logic for my heater
```

Claude will fetch `program.xml` and explain the control logic in plain English.

#### Analyze your water changes

```
> when was my last water change?

I'll check your Apex Neptune aquarium controller data to find
information about your last water change.

Your last water change was yesterday, January 31, 2026, done manually
between approximately 4:58 PM and 5:30 PM.

The Apex Neptune logs show the sequence:
- Return pump and doser were turned off
- Tank was filled with new saltwater (about 13 minutes total)
- Return pump and doser were turned back on

Also, I noticed your Automatic Water Change (AWC) system is currently
set to OFF and hasn't run in at least the past 8 weeks. If you want to
use the automatic system, you'll need to turn the AWC outlet back on.

> and before that?

Before that, you had 6 other manual water changes over the past ~2.5 months:

1. January 11, 2026 - 20 days before (your longest interval)
2. December 31, 2025 - 10 days before that
3. December 12, 2025 - 18 days before that
4. November 29, 2025 - 13 days before that
5. November 22, 2025 - 6 days before that (shortest interval)
6. November 15, 2025 - 7 days before that (earliest in the logs)

Your water change pattern:
- Average interval: 12.3 days
- Shortest: 6 days
- Longest: 20 days (the gap before your most recent change)

All water changes follow the same manual procedure: stir saltwater,
turn off return pump/skimmer/doser, fill tank, restart
equipment. Takes about 10-27 minutes total.
```

This was all extracted directly from historical Apex Neptune logs with no additional context.  The response from the `apex-neptune-retriever` subagent was a much more detailed report which inferred the purpose of each outlet from the name and timing correlations.  Here's an exerpt:

```
===============================================================================
                           STATISTICS SUMMARY
===============================================================================

Total tracking period:         78 days (Nov 15, 2025 - Jan 31, 2026)
Average interval:              12.3 days between water changes
Minimum interval:              6 days (Nov 22 → Nov 29)
Maximum interval:              20 days (Jan 11 → Jan 31)
Typical change duration:       10-27 minutes

Time since last water change:  1 day / 26.5 hours (as of Feb 1, 2026 7:30pm)

===============================================================================
                      WATER CHANGE PROCEDURE PATTERN
===============================================================================

The manual water change procedure follows this consistent pattern:

1. PREPARATION PHASE:
   - SWSTIR (saltwater stirring pump) activated to mix new saltwater
   - Runs for 10-30 minutes before water change begins

2. DRAIN PHASE:
   - RetPump (return pump) turns OFF
   - Skimmer turns OFF  
   - Doser turns OFF
   - Water is presumably siphoned/drained manually (not tracked by Apex)

3. FILL PHASE:
   - TANKFILL outlet activates
   - XFERPUMP (transfer pump) turns ON
   - New saltwater pumped into display tank
   - Multiple fill cycles sometimes used (2-5 minutes each)

4. RESTART PHASE:
   - RetPump turns back ON
   - Doser turns back ON
   - TANKFILL turns OFF
   - Skimmer restarts (5-minute defer period)

5. CLEANUP:
   - Additional XFERPUMP cycles may run
   - System returns to normal operation

===============================================================================
                          EQUIPMENT NOTES
===============================================================================

AWC (Automatic Water Change) Outlet Status:
- Currently: OFF (manually disabled)
- Program: Set ON, auto-shutoff after 90 hours runtime
- Has NOT been active in the past 8+ weeks
- All water changes are being performed MANUALLY

Key Outlets Used During Water Changes:
- RetPump: Main circulation pump (turned off during change)
- Doser: Dosing pump (turned off during change)
- TANKFILL: Fills display tank with new saltwater
- XFERPUMP: Transfer pump (moves water between containers)
- SWSTIR: Stirs/mixes saltwater in preparation container
- Skimmer: Protein skimmer (turned off during change)
```

## Data Sources

The plugin accesses four XML data sources from your Apex Neptune:

| File | Description | Update Frequency |
|------|-------------|------------------|
| `status.xml` | Real-time status with current probe readings and outlet states | Real-time |
| `program.xml` | Current program statements for each outlet | Real-time |
| `datalog.xml` | Historical probe value snapshots | Every 20 minutes (configurable) |
| `outlog.xml` | Outlet state change log | Every state change |

All data files include detailed documentation in the `agents/examples/` directory.

## Network Configuration

By default, the plugin assumes your Apex is accessible at `http://apex.local`. If your controller uses a different hostname or IP address:

1. Edit `/etc/hosts` (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):
   ```
   192.168.1.100    apex.local
   ```

2. Or mention the hostname when asking Claude:
   ```
   Check my tank status at http://apex.home
   ```

## How It Works

This plugin uses a **specialized subagent** called `apex-neptune-retriever` that:

1. Automatically activates when you ask about aquarium parameters
2. Downloads XML data from your Apex Neptune controller
3. Parses and analyzes the sensor readings
4. Presents the information in an easy-to-understand format
5. Provides insights, trends, and alerts based on the data

The subagent runs on the Sonnet model for sophisticated data analysis while keeping costs reasonable.

## Troubleshooting

### Controller Not Found

If Claude can't reach your Apex:
- Verify the controller is powered on and connected to your network
- Check that `apex.local` resolves: `ping apex.local`
- Confirm the web interface is accessible: visit `http://apex.local` in a browser
- Try using the IP address directly if hostname resolution fails

### No Historical Data

If historical queries return no data:
- Verify your Apex logging interval is configured (default: 20 minutes)
- Check that you're requesting data from dates when the controller was online
- Ensure the controller's clock is set correctly

### Permission Issues

If you get permission errors:
- Verify the plugin directory has correct permissions
- Check that Claude Code can access the network
- Ensure no firewall is blocking access to `apex.local`

## Project Structure

```
apex-neptune-claude-plugin/
├── agents/
│   ├── apex-neptune-retriever.md    # Subagent definition
│   └── examples/                    # Example XML files
│       ├── status.xml               # Status example
│       ├── program.xml              # Program example
│       ├── datalog.xml              # Data log example
│       └── outlog.xml               # Outlet log example
├── .claude-plugin/
│   └── plugin.json                  # Plugin metadata
├── LICENSE                          # Apache 2.0 License
└── README.md                        # This file
```

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Author

Don Schwarz ([@schwardo](https://github.com/schwardo))

## Acknowledgments

- Built for [Claude Code](https://claude.com/code) by Anthropic
- Designed for [Apex Neptune](https://www.neptunesystems.com/) aquarium controllers
- Inspired by the reef keeping community

## Links

- [Neptune Systems](https://www.neptunesystems.com/) - Apex Neptune manufacturer
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [Report Issues](https://github.com/schwardo/apex-neptune-claude-plugin/issues)
