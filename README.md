import os
import requests
import json

# Environment Variables for security
NOTION_TOKEN = os.getenv("NOTION_API_KEY")
NOTION_DATABASE_ID = os.getenv("NOTION_DB_ID")
CLICKSIGN_TOKEN = os.getenv("CLICKSIGN_API_KEY")

NOTION_HEADERS = {
    "Authorization": f"Bearer {NOTION_TOKEN}",
    "Content-Type": "application/json",
    "Notion-Version": "2022-06-28"
}

def get_pending_documents_clicksign():
    """
    Fetches documents that are pending signature from the Clicksign API.
    (Used for CNA / Ctrl Play contract management)
    """
    url = f"https://app.clicksign.com/api/v1/documents?access_token={CLICKSIGN_TOKEN}"
    response = requests.get(url)
    
    if response.status_code == 200:
        documents = response.json().get('documents', [])
        # Filter only pending documents
        return [doc for doc in documents if doc.get('status') == 'running']
    else:
        print("Error fetching from Clicksign:", response.status_code)
        return []

def update_notion_status(page_id, status_text):
    """
    Updates the contract status in the Notion CRM.
    """
    url = f"https://api.notion.com/v1/pages/{page_id}"
    
    payload = {
        "properties": {
            "Status da Assinatura": {
                "select": {
                    "name": status_text
                }
            }
        }
    }
    
    response = requests.patch(url, headers=NOTION_HEADERS, data=json.dumps(payload))
    if response.status_code == 200:
        print(f"Successfully updated Notion page {page_id}")
    else:
        print("Failed to update Notion:", response.text)

if __name__ == "__main__":
    print("Starting Clicksign to Notion Sync...")
    # This is a blueprint script. In production, this would run on a scheduler or via Webhook.
    pending_docs = get_pending_documents_clicksign()
    print(f"Found {len(pending_docs)} pending documents.")
