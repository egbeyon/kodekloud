## Using Kube-bench (open-source)

- Installation on linux:
```bash
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz -o kube-bench_0.4.0_linux_amd64.tar.gz
tar -xvf kube-bench_0.4.0_linux_amd64.tar.gz
```
  - link `https://github.com/aquasecurity/kube-bench/releases/tag/v0.4.0`

- Run a kube-bench test
```bash
 ./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml
```


## Task - CIS-CAT Pro Assessor security assessment

We have installed the CIS-CAT Pro Assessor tool called Assessor-CLI, under /root.

Please run the assessment with the Assessor-CLI.sh script inside Assessor directory and generate a report called index.html in the output directory /var/www/html/.Once done, the report can be viewed using the Assessment Report tab located above the terminal.

Run the test in interactive mode and use below settings:

Benchmarks/Data-Stream Collections: : CIS Ubuntu Linux 20.04 LTS Benchmark v2.0.1

Profile : Level 1 - Server

## Solution
- The content of the Assessment-cli.sh file:
```bash
#!/bin/sh

# Absolute path to this script, e.g. /home/user/bin/foo.sh
SCRIPT=$(readlink -f "$0")
# Absolute path this script is in, thus /home/user/bin
SCRIPTPATH=$(dirname "$SCRIPT")

JAVA=java
MAX_RAM_IN_MB=2048
DEBUG=0

which $JAVA 2>&1 > /dev/null

if [ $? -ne "0" ]; then
        echo "Error: Java is not in the system PATH."
        exit 1
fi

JAVA_VERSION_RAW=`$JAVA -version 2>&1`

echo $JAVA_VERSION_RAW | grep 'version\s*\"\(\(1\.8\.\)\|\(9\.\)\|\([1-9][0-9]\.\)\)' 2>&1 > /dev/null

if [ $? -eq "1" ]; then

        echo "Error: The version of Java you are attempting to use is not compatible with CISCAT:"
        echo ""
        echo $JAVA_VERSION_RAW
        echo ""
        echo "You must use Java 1.8.x, or higher. The most recent version of Java is recommended."
        exit 1;
fi

if [ $DEBUG -eq "1" ]; then
        echo "Executing CIS-CAT Pro Assessor from $SCRIPTPATH"
        $JAVA -Xmx${MAX_RAM_IN_MB}M -jar $SCRIPTPATH/Assessor-CLI.jar "$@" --verbose
else
        $JAVA -Xmx${MAX_RAM_IN_MB}M -jar $SCRIPTPATH/Assessor-CLI.jar "$@"
fi
```

- Run this command to generate the report:
```bash
cd /root/Assessor
sh ./Assessor-CLI.sh -i -rd /var/www/html/ -nts -rp index
```
