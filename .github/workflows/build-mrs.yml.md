name: Build MRS



on:

&#x20; push:

&#x20;   branches:

&#x20;     - main

&#x20;   paths:

&#x20;     - "rules/\*.txt"

&#x20;     - ".github/workflows/build-mrs.yml"



&#x20; workflow\_dispatch:



permissions:

&#x20; contents: write



jobs:

&#x20; build:

&#x20;   runs-on: ubuntu-latest



&#x20;   steps:

&#x20;     - name: Checkout

&#x20;       uses: actions/checkout@v4



&#x20;     - name: Get latest Mihomo version

&#x20;       id: mihomo

&#x20;       shell: bash

&#x20;       run: |

&#x20;         VERSION=$(curl -fsSL https://api.github.com/repos/MetaCubeX/mihomo/releases/latest | jq -r '.tag\_name')

&#x20;         echo "version=$VERSION" >> "$GITHUB\_OUTPUT"

&#x20;         echo "Mihomo version: $VERSION"



&#x20;     - name: Download Mihomo

&#x20;       shell: bash

&#x20;       run: |

&#x20;         VERSION="${{ steps.mihomo.outputs.version }}"



&#x20;         wget -q \\

&#x20;           "https://github.com/MetaCubeX/mihomo/releases/download/${VERSION}/mihomo-linux-amd64-${VERSION}.gz" \\

&#x20;           -O mihomo.gz



&#x20;         gunzip mihomo.gz

&#x20;         chmod +x mihomo



&#x20;         ./mihomo -v



&#x20;     - name: Convert rules to MRS

&#x20;       shell: bash

&#x20;       run: |

&#x20;         mkdir -p mrs



&#x20;         for file in rules/\*.txt; do

&#x20;           \[ -e "$file" ] || continue



&#x20;           name=$(basename "$file" .txt)



&#x20;           echo "======================================"

&#x20;           echo "Converting: $file"

&#x20;           echo "Output:     mrs/$name.mrs"

&#x20;           echo "======================================"



&#x20;           ./mihomo convert-ruleset classical text \\

&#x20;             "$file" \\

&#x20;             "mrs/$name.mrs"

&#x20;         done



&#x20;     - name: Commit MRS files

&#x20;       shell: bash

&#x20;       run: |

&#x20;         git config user.name "github-actions\[bot]"

&#x20;         git config user.email "41898282+github-actions\[bot]@users.noreply.github.com"



&#x20;         git add mrs/



&#x20;         if git diff --cached --quiet; then

&#x20;           echo "No MRS changes."

&#x20;           exit 0

&#x20;         fi



&#x20;         git commit -m "Auto build MRS rules"

&#x20;         git push

