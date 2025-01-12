# Revamp Your Terminal: Complete Guide on iTerm2, Tmux and Zsh

After watching several YouTube videos, I decided to completely revamp my terminal setup for a more efficient and visually appealing experience. Here's a breakdown of the tools and steps I followed to achieve my new terminal setup, inspired largely by the YouTube channel TypeCraft.

## Step 1: Install iTerm2

I started by installing [iTerm2](https://iterm2.com/), a powerful alternative to the default macOS terminal. It offers enhanced features like split panes, search, and extensive customization options. I customized the font style and text colors to match my preferences for a more comfortable coding environment.

## Step 2: Install Tmux

Next, I installed [tmux](https://github.com/tmux/tmux), a terminal multiplexer that allows for multiple sessions and panes. Here are some beginner-friendly commands I found useful:

- `tmux list-sessions` – List all current sessions.
- `tmux new-session -s <name>` – Create a new session.
- `tmux attach-session -t <name>` – Attach to an existing session.
- `tmux kill-session -t <name>` – Kill a specific session.

Using tmux helps in managing multiple tasks in a single terminal window, which significantly improves productivity.

## Step 3: Install Oh-My-Zsh

I then switched to [Oh-My-Zsh](https://ohmyz.sh/), a popular framework for managing Zsh configurations. It simplifies managing plugins and themes, making terminal customization easier.

## Step 4: Install Powerlevel10k Theme

To enhance the visual aesthetics further, I installed the [Powerlevel10k](https://github.com/romkatv/powerlevel10k) theme. It provides a highly customizable prompt with icons and color schemes, improving the overall terminal experience.

## Step 5: Enhance Usability with Plugins

I added the following Zsh plugins for better usability:

- `zsh-syntax-highlighting`: Highlights commands as you type for better readability.
- `zsh-autosuggestions`: Provides command suggestions based on your history, saving time while typing.

## Step 6: Install and Alias ColorLS

Finally, I installed [ColorLS](https://github.com/athityakumar/colorls) for a more colorful and informative directory listing. I also created an alias for easier use:

```bash
alias ls='colorls'
```

## Why I Chose These Tools

- **Aesthetics:** A visually pleasing terminal makes the workspace more inviting.
- **Productivity:** Tmux and plugins like autosuggestions improve efficiency.
- **Customization:** iTerm2 and Powerlevel10k offer deep customization to suit individual preferences.

TypeCraft, a YouTube channel focused on terminal optimization, inspired this entire setup.

