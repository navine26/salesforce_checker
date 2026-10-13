import Resolver from '@forge/resolver';
import { fetch, asApp, asUser, route } from '@forge/api';

const resolver = new Resolver();
const N8N_URL = 'https://vignesa.app.n8n.cloud/webhook/Sfview';

resolver.define('getSalesforceData', async (req) => {
  const issueKey = req.context.extension.issue.key;
  const searchTerm = req.payload?.search || '';

  try {
    const response = await fetch(N8N_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        accountNumber: searchTerm,
        issueKey: issueKey,
        objectType: 'Account'
      })
    });
    if (!response.ok) return { error: `n8n error: ${response.status}` };
    return await response.json();
  } catch (error) {
    return { error: 'Failed to reach n8n.' };
  }
});

// Fetch a single Salesforce Case via the same n8n webhook.
// Expected payload once n8n is updated: objectType Case + caseNumber (+ accountNumber for context).
resolver.define('getCaseDetail', async (req) => {
  const issueKey = req.context.extension.issue.key;
  const caseNumber = req.payload?.caseNumber || '';
  const accountNumber = req.payload?.accountNumber || '';

  if (!caseNumber) {
    return { error: 'Missing caseNumber' };
  }

  try {
    const response = await fetch(N8N_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        accountNumber,
        caseNumber,
        issueKey,
        objectType: 'Case'
      })
    });
    if (!response.ok) return { error: `n8n error: ${response.status}` };
    const json = await response.json();
    // Empty body / empty object from n8n
    if (!json || (typeof json === 'object' && !json.data && !json.found && Object.keys(json).length === 0)) {
      return { error: 'No case detail returned from n8n.', caseNumber };
    }
    return json;
  } catch (error) {
    return { error: 'Failed to reach n8n for case detail.' };
  }
});

resolver.define('getIssueField', async (req) => {
  const issueKey = req.context.extension.issue.key;
  const response = await asUser().requestJira(route`/rest/api/3/issue/${issueKey}?fields=customfield_12911`);
  const data = await response.json();
  const fieldVal = data.fields?.customfield_12911;

  if (Array.isArray(fieldVal)) {
    return fieldVal[0] || null;
  }
  return fieldVal || null;
});

resolver.define('linkAccountToIssue', async (req) => {
  const issueKey = req.context.extension.issue.key;
  const { accountNumber } = req.payload;

  const response = await asApp().requestJira(route`/rest/api/3/issue/${issueKey}`, {
    method: 'PUT',
    headers: { 'Accept': 'application/json', 'Content-Type': 'application/json' },
    body: JSON.stringify({
      fields: {
        customfield_12911: accountNumber
      }
    })
  });
  return { success: response.status === 204 };
});

export const handler = resolver.getDefinitions();
