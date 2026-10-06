# Pax
SI Assistant 
steps:
  - name: Check out repository
    uses: actions/checkout@v4

  - name: Validate .gitignore contains KiCad temp file exclusions
    shell: bash
    run: |
      set -euo pipefail

      required_patterns=(
        '*.000'
        '*.bak'
        '*.bck'
        '*.kicad_pcb-bak'
        '*.kicad_sch-bak'
        '*-backups'
        '*-cache*'
        '*-bak'
        '*-bak*'
        '*~'
        '~*'
        '\#auto_saved_files\#'
        '*.tmp'
        '*-save.pro'
        '*-save.kicad_pcb'
        'fp-info-cache'
        '~*.lck'
        '*.net'
        '*.dsn'
        '*.ses'
        '*.xml'
        '*.csv'
        '.history'
        '*.kicad_prl'
      )

      missing=0
      for pattern in "${required_patterns[@]}"; do
        if ! grep -Fq "$pattern" .gitignore; then
          echo "Missing required ignore pattern: $pattern"
          missing=1
        fi
      done

      if [ "$missing" -ne 0 ]; then
        echo "One or more KiCad ignore rules are missing from .gitignore."
        exit 1
      fi

  - name: Fail if generated KiCad files are tracked in git
    shell: bash
    run: |
      set -euo pipefail

      tracked_files=$(git ls-files)
      bad_files=""

      while IFS= read -r file; do
        case "$file" in
          *.000|*.bak|*.bck|*.kicad_pcb-bak|*.kicad_sch-bak|*-backups|*-cache*|*-bak|*-bak*|*~|~*|*.tmp|*-save.pro|*-save.kicad_pcb|~*.lck|*.net|*.dsn|*.ses|*.xml|*.csv|*.kicad_prl|.history)
            bad_files+="$file