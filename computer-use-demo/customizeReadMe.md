```markdown:customizeReadme.md
# Customizing the Computer Use Demo Docker Environment

This guide explains how to customize the Docker environment for the Anthropic Computer Use Demo.

## Table of Contents
- [Screen Resolution](#screen-resolution)
- [Desktop Environment](#desktop-environment)
- [Applications](#applications)
- [Python Environment](#python-environment)
- [System Dependencies](#system-dependencies)

## Screen Resolution

The Docker container supports customizable screen resolutions through environment variables:
- `WIDTH`: Set the screen width (default: 1024)
- `HEIGHT`: Set the screen height (default: 768)

Example usage:
```bash
docker run \
    -e WIDTH=1920 \
    -e HEIGHT=1080 \
    [other docker options...]
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

**Note**: It's recommended to stay within XGA resolution (1024x768) for optimal performance. The system will automatically handle scaling for higher resolutions.

## Desktop Environment

The container uses a minimal desktop environment with tint2 as the panel/taskbar. You can customize:

1. Panel Layout: Edit `.config/tint2/tint2rc`
   - Panel position
   - Size and padding
   - Background colors
   - Taskbar behavior

2. Application Launchers: Modify the launcher items in `.config/tint2/tint2rc`
   - Add/remove application shortcuts
   - Change icon sizes
   - Customize launcher padding

## Applications

### Default Applications
The environment comes with these pre-installed applications:
- Firefox ESR
- LibreOffice Calc
- XPaint
- XPDF
- Gedit
- Galculator
- Terminal

### Adding New Applications

1. Install via Dockerfile:
```dockerfile
RUN apt-get update && apt-get install -y \
    your-package-name \
    another-package
```

2. Create a desktop entry in `.config/tint2/applications/`:
```ini
[Desktop Entry]
Name=Application Name
Comment=Description
Exec=command-to-run
Icon=icon-name
Terminal=false
Type=Application
Categories=Category;
```

3. Add the launcher entry to `.config/tint2/tint2rc`:
```
launcher_item_app = /path/to/your/application.desktop
```

## Python Environment

The container uses pyenv to manage Python versions. To customize:

1. Change Python version:
   - Modify these environment variables in the Dockerfile:
     ```dockerfile
     ENV PYENV_VERSION_MAJOR=3
     ENV PYENV_VERSION_MINOR=11
     ENV PYENV_VERSION_PATCH=6
     ```

2. Add Python packages:
   - Modify `computer_use_demo/requirements.txt`
   - Or add additional pip install commands to the Dockerfile

## System Dependencies

The base system uses Ubuntu 22.04. To add system dependencies:

1. Add packages to the main apt-get install command in the Dockerfile:
```dockerfile
RUN apt-get update && apt-get install -y \
    your-new-package \
    another-package
```

2. For packages from PPAs:
```dockerfile
RUN add-apt-repository ppa:your-ppa && \
    apt-get update && \
    apt-get install -y package-name
```

## Development Tips

1. For local development, mount your local directory:
```bash
docker run \
    -v $(pwd)/computer_use_demo:/home/computeruse/computer_use_demo/ \
    [other options...]
    -it computer-use-demo:local
```

2. Use the provided setup script for development environment:
```bash
./setup.sh
```

3. Changes to the desktop environment configuration can be tested by rebuilding the Docker image:
```bash
docker build . -t computer-use-demo:local
```
```

This documentation references the following code sections:


```1:52:Dockerfile
FROM docker.io/ubuntu:22.04
# Non-interactive frontend
ENV DEBIAN_FRONTEND=noninteractive
ENV DEBIAN_PRIORITY=high

RUN apt-get update && \
    apt-get -y upgrade && \
    apt-get -y install \
    # UI Requirements
    xvfb \
    xterm \
    xdotool \
    scrot \
    imagemagick \
    sudo \
    mutter \
    x11vnc \
    # Python/pyenv reqs
    build-essential \
    libssl-dev  \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    curl \
    git \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libffi-dev \
    liblzma-dev \
    # Network tools
    net-tools \
    netcat \
    # PPA req
    software-properties-common && \
    # Userland apps
    sudo add-apt-repository ppa:mozillateam/ppa && \
    sudo apt-get install -y --no-install-recommends \
    libreoffice \
    firefox-esr \
    x11-apps \
    xpdf \
    gedit \
    xpaint \
    tint2 \
    galculator \
    pcmanfm \
    unzip && \
    apt-get clean
```



```75:88:Dockerfile
ENV PYENV_VERSION_MAJOR=3
ENV PYENV_VERSION_MINOR=11
ENV PYENV_VERSION_PATCH=6
ENV PYENV_VERSION=$PYENV_VERSION_MAJOR.$PYENV_VERSION_MINOR.$PYENV_VERSION_PATCH
RUN eval "$(pyenv init -)" && \
    pyenv install $PYENV_VERSION && \
    pyenv global $PYENV_VERSION && \
    pyenv rehash

ENV PATH="$HOME/.pyenv/shims:$HOME/.pyenv/bin:$PATH"

RUN python -m pip install --upgrade pip==23.1.2 setuptools==58.0.4 wheel==0.40.0 && \
    python -m pip config set global.disable-pip-version-check true

```


The desktop environment configuration can be found in:


```1:100:image/.config/tint2/tint2rc
#-------------------------------------
# Panel
panel_items = TL
panel_size = 100% 60
panel_margin = 0 0
panel_padding = 2 0 2
panel_background_id = 1
wm_menu = 0
panel_dock = 0
panel_position = bottom center horizontal
panel_layer = top
panel_monitor = all
panel_shrink = 0
autohide = 0
autohide_show_timeout = 0
autohide_hide_timeout = 0.5
autohide_height = 2
strut_policy = follow_size
panel_window_name = tint2
disable_transparency = 1
mouse_effects = 1
font_shadow = 0
mouse_hover_icon_asb = 100 0 10
mouse_pressed_icon_asb = 100 0 0
scale_relative_to_dpi = 0
scale_relative_to_screen_height = 0

#-------------------------------------
# Taskbar
taskbar_mode = single_desktop
taskbar_hide_if_empty = 0
taskbar_padding = 0 0 2
taskbar_background_id = 0
taskbar_active_background_id = 0
taskbar_name = 1
taskbar_hide_inactive_tasks = 0
taskbar_hide_different_monitor = 0
taskbar_hide_different_desktop = 0
taskbar_always_show_all_desktop_tasks = 0
taskbar_name_padding = 4 2
taskbar_name_background_id = 0
taskbar_name_active_background_id = 0
taskbar_name_font_color = #e3e3e3 100
taskbar_name_active_font_color = #ffffff 100
taskbar_distribute_size = 0
taskbar_sort_order = none
task_align = left

#-------------------------------------
# Launcher
launcher_padding = 4 8 4
launcher_background_id = 0
launcher_icon_background_id = 0
launcher_icon_size = 48
launcher_icon_asb = 100 0 0
launcher_icon_theme_override = 0
startup_notifications = 1
launcher_tooltip = 1

#-------------------------------------
# Launcher icon
launcher_item_app = /usr/share/applications/libreoffice-calc.desktop
launcher_item_app = /home/computeruse/.config/tint2/applications/terminal.desktop
launcher_item_app = /home/computeruse/.config/tint2/applications/firefox-custom.desktop
launcher_item_app = /usr/share/applications/xpaint.desktop
launcher_item_app = /usr/share/applications/xpdf.desktop
launcher_item_app = /home/computeruse/.config/tint2/applications/gedit.desktop
launcher_item_app = /usr/share/applications/galculator.desktop

#-------------------------------------
# Background definitions
# ID 1
rounded = 0
border_width = 0
background_color = #000000 60
border_color = #000000 30

# ID 2
rounded = 4
border_width = 1
background_color = #777777 20
border_color = #777777 30

# ID 3
rounded = 4
border_width = 1
background_color = #777777 20
border_color = #ffffff 40

# ID 4
rounded = 4
border_width = 1
background_color = #aa4400 100
border_color = #aa7733 100

# ID 5
rounded = 4
border_width = 1
background_color = #aaaa00 100
border_color = #aaaa00 100
```


And the Firefox desktop entry example:


```1:9:image/.config/tint2/applications/firefox-custom.desktop
[Desktop Entry]
Name=Firefox Custom
Comment=Open Firefox with custom URL
Exec=firefox-esr -new-window
Icon=firefox-esr
Terminal=false
Type=Application
Categories=Network;WebBrowser;

```
