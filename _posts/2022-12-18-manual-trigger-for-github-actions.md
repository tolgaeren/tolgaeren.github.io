# [DRAFT] Manually triggering GitHub actions to run scripts with environment scripts

For an github action to be manually triggerable, it must have on `workflow_dispatch` set:
> `.github/workflows/clear_consults.yml`
>```yaml
>
>on: 
>  workflow_dispatch:
>    inputs:
>      deployment:
>        type: environment  # show a list of available environments
>        description: Which deployment to clear consults from
>
>jobs:
>  clear_test_db:
>    name: Clear Consults
>    environment: ${{ github.event.inputs.deployment }}  # set the user selected Github Environment
>    runs-on: ubuntu-20.04
>    steps:
>      - uses: actions/checkout@v3
>      - name: Install dependencies # install gettext-base for `envsubst`
>        run: | 
>          sudo apt-get update && sudo apt-get install -y gettext-base 
>        shell: bash
>      - name: Run script file
>        env:
>            BASE_URL: ${{ secrets.BASE_URL }} # secrets come from the environment
>            USER_ROLE: ${{ secrets.USERNAME }}
>            ADMIN_SECRET: ${{ secrets.USER_SECRET }}
>        run: | # replace these variables from the script with the Environment secrets
>          envsubst '$BASE_URL,$USERNAME,$USER_SECRET' < script.sh.template > script.sh
>          chmod +x script.sh
>          ./script.sh
>        shell: bash
>```


example script:

>`script.template.sh` 
>```bash
>#!/bin/bash
>curl $BASE_URL/api/v1/items/DeleteAll \
>    -u "$USER_NAME:$USER_SECRET"
>```