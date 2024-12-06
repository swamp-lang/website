# Official Website for Swamp Language

## Setup Instructions

1. Install Zola:
   - **macOS**: `brew install zola`
   - **Linux**: 
     - Ubuntu/Debian: `sudo apt install zola`
     - Fedora: `sudo dnf install zola`
     - Arch: `sudo pacman -S zola`
   - **Windows**: Download from [Zola releases](https://github.com/getzola/zola/releases)

   More information can be found [here](https://www.getzola.org/documentation/getting-started/installation/).

## Development

To work on the website locally:

1. Clone this repository
2. Navigate to the project directory
3. Run `zola serve`
4. Open http://127.0.0.1:1111 in your browser

The site will automatically rebuild and refresh when you make changes to the source files.

## Building

To build the static site:

1. Run `zola build`
2. The generated site will be in the `output/` directory

Note: The `output/` directory is automatically generated and should not be committed to the repository. It's already included in `.gitignore`. If you're deploying the site manually, use the contents of this directory.