Create  ECS Cluster:

[mentor@vagrant ~]$ aws ecs create-cluster --cluster-name Jenkins
{
    "cluster": {
        "clusterArn": "arn:aws:ecs:us-east-1:264112291867:cluster/Jenkins",
        "clusterName": "Jenkins",
        "status": "ACTIVE",
        "registeredContainerInstancesCount": 0,
        "runningTasksCount": 0,
        "pendingTasksCount": 0,
        "activeServicesCount": 0,
        "statistics": [],
        "tags": []
    }
}


