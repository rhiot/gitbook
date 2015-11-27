# Manage Kura Maven dependencies

Kura Platform comes with lot of module and artifact. Rhiot project has to synchronize correct module and artifact version into BOM file.

## To sync Kura module version

Execute Bash script below and copy result and replace it into the [BOM pom.xml file](https://github.com/rhiot/rhiot/blob/master/bom/pom.xml)

```
for i in `cat  distrib/config/kura.build.properties ` ; do

artifcatId=$(echo $i | cut -f 1 -d = )
versionId=$(echo $i | cut -f 2 -d = )

  echo "<$artifcatId>$versionId</$artifcatId>"

done
```

## To sync Kura module dependencies

Execute Bash script below and copy result and replace it into the [BOM pom.xml file](https://github.com/rhiot/rhiot/blob/master/bom/pom.xml)

```
for i in `cat  distrib/config/kura.build.properties ` ; do

artifcatId=$(echo $i | cut -f 1 -d = | sed -e 's#.version##' )
versionId=$(echo $i | cut -f 2 -d = )

 echo "<dependency><groupId>org.eclipse.kura</groupId><artifactId>$artifcatId</artifactId><version>\${$artifcatId.version}</version>  </dependency>"

done
```

