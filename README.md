# Arquitetura-middleware
Implementação 
name: flake8

on:
  pull_request:
    types:
      - 'synchronize'
      - 'opened'

jobs:
  lint:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python 3.9
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install flake8
    - name: Get current errors
      run: |
        tmpafter=$(mktemp)
        find src -name \*.py -exec flake8 --ignore=E402,E501,W504 {} + | egrep -v "alembic/versions/|usr/local/share/pysnmp/mibs/" > $tmpafter
        num_errors_after=`cat $tmpafter | wc -l`
        echo "CURRENT_ERROR_FILE=${tmpafter}" >> $GITHUB_ENV
        echo "CURRENT_ERRORS=${num_errors_after}" >> $GITHUB_ENV
    - name: Checkout base branch
      uses: actions/checkout@v2
      with:
        ref: ${{ github.base_ref }}
    - name: Get errors from base branch
      run: |
        tmpbefore=$(mktemp)
        find src -name \*.py -exec flake8 --ignore=E402,E501,W504 {} + | egrep -v "alembic/versions/|usr/local/share/pysnmp/mibs/" > $tmpbefore
        num_errors_before=`cat $tmpbefore | wc -l`
        echo "OLD_ERROR_FILE=${tmpbefore}" >> $GITHUB_ENV
        echo "OLD_ERRORS=${num_errors_before}" >> $GITHUB_ENV
    - name: Conclusion
      run: |
        if [ ${{ env.CURRENT_ERRORS }} -gt ${{ env.OLD_ERRORS }} ]; then
          echo "New flake8 errors were introduced"
          diff -u ${{ env.OLD_ERROR_FILE }} ${{ env.CURRENT_ERROR_FILE }}
          exit 1
        else
          echo "No new flake 8 errors were introduced"
        fi
