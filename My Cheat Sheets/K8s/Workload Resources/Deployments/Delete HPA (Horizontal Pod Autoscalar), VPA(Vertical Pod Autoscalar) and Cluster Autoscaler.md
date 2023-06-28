To delete an autoscaler in Kubernetes using `kubectl`, you can use the `kubectl delete` command with the appropriate resource name. The specific command depends on the type of autoscaler you want to delete. Here are the commands for deleting different types of autoscalers:

1. Horizontal Pod Autoscaler (HPA):
   Use the following command to delete an HPA:
   ```
   kubectl delete hpa <hpa-name>
   ```

   Replace `<hpa-name>` with the name of the HPA you want to delete.

2. Vertical Pod Autoscaler (VPA):
   Use the following command to delete a VPA:
   ```
   kubectl delete vpa <vpa-name>
   ```

   Replace `<vpa-name>` with the name of the VPA you want to delete.

3. Cluster Autoscaler (CA):
   Use the following command to delete the Cluster Autoscaler:
   ```
   kubectl delete clusterautoscaler <ca-name> --namespace=<namespace>
   ```

   Replace `<ca-name>` with the name of the Cluster Autoscaler, and `<namespace>` with the namespace where the Cluster Autoscaler is deployed.

Remember to replace `<hpa-name>`, `<vpa-name>`, `<ca-name>`, and `<namespace>` with the actual names and namespace of the autoscaler you want to delete.

Make sure you have the necessary permissions to delete the autoscaler, and ensure that you are using the correct `kubectl` context and cluster configuration.