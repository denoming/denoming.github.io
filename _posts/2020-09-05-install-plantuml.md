---
title: "Install PlantUML"
date: 2020-09-05
categories: [Development,Tools]
tags: [plantuml]
---

## Install

Install prerequisites:
```bash
$ sudo apt install -y curl graphviz default-jre
```

Install PlantUML binary:
```bash
$ sudo mkdir -p /opt/plantuml
$ cd /opt/plantuml
$ UML=http://sourceforge.net/projects/plantuml/files/plantuml.jar/download
$ sudo curl -JLO ${UML}
```

Create runner script:
```bash
$ cat <<EOF | sudo tee /usr/local/bin/plantuml
#!/bin/sh
java -jar /opt/plantuml/plantuml.jar "\$@"
EOF
$ sudo chmod a+x /usr/local/bin/plantuml
```

## Use

Create diagram file _sequence.puml_:
```bash
cat <<EOF > sequence.pu
@startuml
Alice -> Bob: test
@enduml
EOF
```

Convert to diagram:
```bash
$ plantuml sequence.pu
```



