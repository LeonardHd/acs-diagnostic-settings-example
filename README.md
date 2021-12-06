# ACS Azure Policies:

1. Option - Use Built in `Audit diagnostic setting` Azure Policy via Portal or other management tool.

    Also see [GitHub](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/DiagnosticSettingsForTypes_Audit.json)

2. Option - Use Custom Audit policy for specific logs / metrics

    Defintion:
    ```json
    {
        "properties": {
            "displayName": "ACS Audit Existing Diagnostic Settings Metrics (Custom) Policy",
            "policyType": "Custom",
            "mode": "All",
            "metadata": {
                "category": "Custom"
            },
            "parameters": {},
            "policyRule": {
                "if": {
                    "field": "type",
                    "equals": "Microsoft.Communication/communicationServices"
                },
                "then": {
                    "effect": "AuditIfNotExists",
                    "details": {
                        "type": "Microsoft.Insights/diagnosticSettings",
                        "existenceCondition": {
                            "allOf": [
                                {
                                    "field": "Microsoft.Insights/diagnosticSettings/logs.enabled",
                                    "equals": "true"
                                },
                                {
                                    "field": "Microsoft.Insights/diagnosticSettings/metrics[*].enabled",
                                    "equals": "true"
                                },
                                {
                                    "field": "Microsoft.Insights/diagnosticSettings/metrics[*].category",
                                    "equals": "Traffic"
                                }
                            ]
                        }
                    }
                }
            }
        }
    }
    ```

3. Option - Use Custom DeployIfNotExists Policy

    ```json
    {
        "properties": {
            "displayName": "ACS Deploy Diagnostic Settings (Custom) Policy",
            "policyType": "Custom",
            "mode": "All",
            "metadata": {
                "category": "Custom"
            },
            "parameters": {
                "diagnosticSettingsName": {
                    "type": "String",
                    "metadata": {
                        "displayName": "diagnosticSettingsName",
                        "description": "Name of diagnostic settings"
                    }
                },
                "logAnalyticsInstance": {
                    "type": "String",
                    "metadata": {
                        "displayName": "Log Analystics",
                        "description": "Target Log Analystics Workspace",
                        "strongType": "omsWorkspace"
                    }
                }
            },
            "policyRule": {
                "if": {
                    "field": "type",
                    "equals": "Microsoft.Communication/communicationServices"
                },
                "then": {
                    "effect": "DeployIfNotExists",
                    "details": {
                        "type": "Microsoft.Insights/diagnosticSettings",
                        "name": "[parameters('diagnosticSettingsName')]",
                        "existenceCondition": {
                            "allOf": [
                                {
                                    "count": {
                                        "field": "Microsoft.Insights/diagnosticSettings/logs[*]",
                                        "where": {
                                            "allOf": [
                                                {
                                                    "field": "Microsoft.Insights/diagnosticSettings/logs[*].category",
                                                    "equals": "AuthOperational"
                                                },
                                                {
                                                    "field": "Microsoft.Insights/diagnosticSettings/logs[*].enabled",
                                                    "equals": true
                                                }
                                            ]
                                        }
                                    },
                                    "equals": 1
                                },
                                {
                                    "count": {
                                        "field": "Microsoft.Insights/diagnosticSettings/metrics[*]",
                                        "where": {
                                            "allOf": [
                                                {
                                                    "field": "Microsoft.Insights/diagnosticSettings/metrics[*].category",
                                                    "equals": "Traffic"
                                                },
                                                {
                                                    "field": "Microsoft.Insights/diagnosticSettings/metrics[*].enabled",
                                                    "equals": true
                                                }
                                            ]
                                        }
                                    },
                                    "equals": 1
                                }
                            ]
                        },
                        "roleDefinitionIds": [
                            "/providers/microsoft.authorization/roleDefinitions/749f88d5-cbae-40b8-bcfc-e573ddc772fa",
                            "/providers/microsoft.authorization/roleDefinitions/92aaf0da-9dab-42b6-94a3-d43ce8d16293"
                        ],
                        "deployment": {
                            "properties": {
                                "mode": "incremental",
                                "template": {
                                    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                                    "contentVersion": "1.0.0.0",
                                    "parameters": {
                                        "resourceName": {
                                            "type": "string"
                                        },
                                        "location": {
                                            "type": "string"
                                        },
                                        "logAnalyticsInstance": {
                                            "type": "string"
                                        },
                                        "diagnosticSettingsName": {
                                            "type": "string"
                                        }
                                    },
                                    "variables": {},
                                    "resources": [
                                        {
                                            "type": "Microsoft.Communication/communicationServices/providers/diagnosticSettings",
                                            "apiVersion": "2021-05-01-preview",
                                            "name": "[concat(parameters('resourceName'), '/', 'Microsoft.Insights/', parameters('diagnosticSettingsName'))]",
                                            "location": "[parameters('location')]",
                                            "dependsOn": [],
                                            "properties": {
                                                "workspaceId": "[parameters('logAnalyticsInstance')]",
                                                "metrics": [
                                                    {
                                                        "category": "Traffic",
                                                        "enabled": true,
                                                        "retentionPolicy": {
                                                            "enabled": false,
                                                            "days": 31
                                                        }
                                                    }
                                                ],
                                                "logs": [
                                                    {
                                                        "category": "AuthOperational",
                                                        "enabled": true
                                                    }
                                                ]
                                            }
                                        }
                                    ],
                                    "outputs": {}
                                },
                                "parameters": {
                                    "resourceName": {
                                        "value": "[field('name')]"
                                    },
                                    "location": {
                                        "value": "[field('location')]"
                                    },
                                    "logAnalyticsInstance": {
                                        "value": "[parameters('logAnalyticsInstance')]"
                                    },
                                    "diagnosticSettingsName": {
                                        "value": "[parameters('diagnosticSettingsName')]"
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    ```
