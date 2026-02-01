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

Clone this repository and install it as a Claude Code plugin:

```bash
# Clone the repository
git clone https://github.com/schwardo/apex-neptune-claude-plugin.git
cd apex-neptune-claude-plugin

# Install as a plugin (symlink to Claude Code plugins directory)
mkdir -p ~/.claude/plugins
ln -s "$(pwd)" ~/.claude/plugins/apex-neptune
```

Alternatively, if you prefer to copy instead of symlink:

```bash
cp -r apex-neptune-claude-plugin ~/.claude/plugins/apex-neptune
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
