# Job Apply Plugin for Claude Code

AI-powered job application assistant that automatically fills out job applications on LinkedIn Easy Apply, Greenhouse, Ashby, and Workday using browser automation.

## Features

- **One-time profile setup**: Extract your information from a resume (PDF, DOCX, or TXT)
- **Multi-platform support**: LinkedIn Easy Apply, Greenhouse, Ashby, Workday
- **Smart field mapping**: Automatically matches your profile to form fields
- **Safety first**: Never submits without your explicit confirmation
- **Resume storage**: Profile saved locally for reuse across applications

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome) MCP server for browser automation

## Installation

### From GitHub Marketplace

```bash
/plugin marketplace add neonwatty/job-apply-plugin
/plugin install job-apply
```

### Manual Installation

Clone this repository to your local machine:

```bash
git clone https://github.com/neonwatty/job-apply-plugin.git
cd job-apply-plugin
```

Then in Claude Code:
```bash
/plugin install /path/to/job-apply-plugin
```

### Claude Desktop Installation

Claude Desktop requires manual skill upload:

1. **Download the ZIP file** from the [latest release](https://github.com/neonwatty/job-apply-plugin/releases/latest)

2. **Open Claude Desktop** and go to **Settings**

3. **Click "Upload skill"** and select the downloaded `.zip` file

4. **Restart Claude Desktop** to load the skill

> **Note:** Claude Desktop does not have marketplace support. You'll need to manually download updates from the releases page.

## Usage

### First Time Setup

1. Invoke the skill:
   ```
   /job-apply
   ```

2. Provide your resume path when prompted:
   ```
   ~/Documents/resume.pdf
   ```

3. Review and confirm extracted profile information

### Applying to Jobs

Once your profile is set up:

1. Invoke the skill:
   ```
   /job-apply
   ```

2. Provide a job URL:
   ```
   https://www.linkedin.com/jobs/view/123456789
   ```

3. Watch as Claude fills out the application

4. Review the summary and confirm submission

## Supported Platforms

| Platform | URL Pattern | Status |
|----------|-------------|--------|
| LinkedIn Easy Apply | `linkedin.com/jobs/view/*` | Supported |
| Greenhouse | `boards.greenhouse.io/*` | Supported |
| Ashby | `jobs.ashbyhq.com/*` | Supported |
| Workday | `*.myworkdayjobs.com/*` | Supported |
| Lever | `jobs.lever.co/*` | Supported |

## Profile Storage

Your profile is stored at `~/.claude-job-profile.json` and includes:

- Personal information (name, email, phone, location)
- Work history
- Education
- Skills
- Social links (LinkedIn, GitHub, portfolio)

To reset your profile:
```
/job-apply reset profile
```

## Safety Features

- **Never enters passwords** - Stops if login is required
- **Never creates accounts** - You must create accounts yourself
- **Never submits without confirmation** - Always shows summary first
- **Never enters payment info** - Skips premium features
- **Confirms sensitive questions** - Salary, visa status, etc.

## License

MIT License - See [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome! Please open an issue or PR on GitHub.

## Author

Jeremy Watt ([@neonwatty](https://github.com/neonwatty))
