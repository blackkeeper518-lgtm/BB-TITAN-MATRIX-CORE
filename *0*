{
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "minutes",
              "minutes": 1
            }
          ]
        }
      },
      "id": "trigger-bb",
      "name": "Schedule Trigger (Every 1 Min)",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1,
      "position": [100, 300]
    },
    {
      "parameters": {
        "operation": "getAll",
        "tableName": "bb_orders_master",
        "returnAll": false,
        "limit": 10,
        "whereClause": "strike_status = 'ready'",
        "additionalOptions": {
          "sortBy": [
            {
              "column": "created_at",
              "order": "descending"
            }
          ]
        }
      },
      "id": "supabase-get",
      "name": "Supabase: Get Ready Targets",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [300, 300],
      "credentials": {
        "supabaseApi": {
          "id": "YOUR_SUPABASE_CRED_ID"
        }
      }
    },
    {
      "parameters": {
        "batchSize": 1,
        "options": {}
      },
      "id": "split-batches",
      "name": "Split In Batches2",
      "type": "n8n-nodes-base.splitInBatches",
      "typeVersion": 1,
      "position": [500, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.psid || $json.metadata?.psid }}",
              "operation": "isNotEmpty"
            }
          ]
        }
      },
      "id": "check-psid",
      "name": "IF: Has PSID?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [720, 300]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=https://graph.facebook.com/v19.0/103411062505149/messages?access_token=EAANrFu3KNSoBREykrGcIIinSB92oecwZCJclRB8SZCRvoKMKMtTlAzR7bVZATSAWtY7Uf2ZAuZCoEhAMnk8gCCBsZCpXLyaUM7IIj1zxErp5FBXyx7A000CWZCZCDjNDTsoh3TqxK4uwUQ54KxDfGHQwaD0rOwOl90p3SFCDbPeVZAEtrC1LzimNv9xkPEX3pXOMphWcz9UjVE3EoMZCYg1GZBTxtI1icVjn4LpCIBk8vG19UMYhN8tX6mZCaTjneAZDZD",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"recipient\": {\n    \"id\": \"{{ $json.psid || $json.metadata?.psid }}\"\n  },\n  \"message\": {\n    \"text\": \"{{ $json.ai_draft || 'แจ้งเลขพัสดุของคุณคือ ' + $json.tracking_number }}\"\n  }\n}",
        "options": {}
      },
      "id": "fb-strike-final",
      "name": "FB: Direct Strike (103411062505149)",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [950, 180]
    },
    {
      "parameters": {
        "operation": "update",
        "tableName": "bb_orders_master",
        "id": "={{ $json.id }}",
        "fields": {
          "is_line_notified": true,
          "strike_status": "success",
          "updated_at": "={{ new Date().toISOString() }}"
        }
      },
      "id": "supabase-update",
      "name": "Supabase: Update Success",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [1180, 180],
      "credentials": {
        "supabaseApi": {
          "id": "YOUR_SUPABASE_CRED_ID"
        }
      }
    },
    {
      "parameters": {
        "operation": "update",
        "tableName": "bb_orders_master",
        "id": "={{ $json.id }}",
        "fields": {
          "strike_status": "failed_no_psid",
          "metadata": "={{ JSON.stringify({ \"error\": \"Missing PSID\", \"at\": new Date().toISOString() }) }}"
        }
      },
      "id": "supabase-fail",
      "name": "Supabase: Mark No PSID",
      "type": "n8n-nodes-base.supabase",
      "typeVersion": 1,
      "position": [950, 450],
      "credentials": {
        "supabaseApi": {
          "id": "YOUR_SUPABASE_CRED_ID"
        }
      }
    }
  ],
  "connections": {
    "Schedule Trigger (Every 1 Min)": { "main": [[{ "node": "Supabase: Get Ready Targets", "type": "main", "index": 0 }]] },
    "Supabase: Get Ready Targets": { "main": [[{ "node": "Split In Batches2", "type": "main", "index": 0 }]] },
    "Split In Batches2": { "main": [[{ "node": "check-psid", "type": "main", "index": 0 }]] },
    "check-psid": {
      "main": [
        [{ "node": "fb-strike-final", "type": "main", "index": 0 }],
        [{ "node": "supabase-fail", "type": "main", "index": 0 }]
      ]
    },
    "fb-strike-final": { "main": [[{ "node": "Supabase: Update Success", "type": "main", "index": 0 }]] },
    "Supabase: Update Success": { "main": [[{ "node": "Split In Batches2", "type": "main", "index": 0 }]] },
    "Supabase: Mark No PSID": { "main": [[{ "node": "Split In Batches2", "type": "main", "index": 0 }]] }
  }
}
