# Kubernetes Carbon Intensity Exporter

This repo provides a data exporter by which Kubernetes operators can leverage the carbon intensity data from 3rd party for carbon-aware workload scheduling.

## Installation

We provide a helm chart to help install the exporter. Note that this data exporter ONLY retrieves the carbon intensity data from 
[WattTime](https://www.watttime.org/). You need to get the **authentication ID/Password** from WattTime organization before using the exporter.

```bash
export WTUSERNAME=XXXX   # WattTime auth info.
export WTPASSWORD=YYYY
export REGION=eastus     # The region where the AKS cluster locates.

helm install carbon-intensity-exporter \
   --set carbonDataExporter.region=$REGION \
   --set apiServer.username=$WTUSERNAME \
   --set apiServer.password=$WTPASSWORD \
   ./charts/carbon-intensity-exporter
```
You should be able to see one exporter Pod running in the `kube-system` namespace.
```bash
$ kubectl get pod -n kube-system | grep carbon-e2e-carbon-intensity-exporter
$ carbon-e2e-carbon-intensity-exporter-XXXXXXX-XXXXX   2/2     Running   0          3m25s
```

You should also see one configmap `carbon-intensity` is created in the `kube-system` namespace.
```bash
$ kubectl get configmap -n kube-system | grep carbon-intensity
$ carbon-intensity                        7      3m25s
```

## Integration
The configmap is formatted as the following:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: carbonintensity
  namespace: kube-system
immutable: true
data:
  lastHeartbeatTime: # The latest time that the data exporter controller sends the data. 
  message: # Additional information for user notification, if any. 
  numOfRecords: # The number can be any value between 0 (no records for the current location) and 24 * 12. 
  forecastDateTime: # The time when the raw data was generated.
  minForcast: # min forecast in the data.
  maxForcast: # max forecast in the data.
binarydata: 
  data: # json marshal of the EmissionsData array.
```

The EmissionData struct is defined in [here](pkg/sdk/api/emissions_data.go). The data exporter will retrieve the 24-hour carbon intensity forecast data
from WattTime every 12 hours. Upon successful data pull, the old configmap will be deleted and a new configmap with the same name will be created.
If the data pull hits failures, the new confgimap is still created with the last seen binary data and the failure reason should be mentioned in the value
of the `message` key. Any Kubernetes operator can read the configmap for utilizing the carbon intensity data.


## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
