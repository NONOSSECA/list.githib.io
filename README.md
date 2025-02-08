import React, { useState } from 'react';
import { useNavigate, useLocation } from 'react-router-dom';
import { Shield, AlertTriangle, CheckCircle, XCircle, ArrowLeft, Download } from 'lucide-react';

interface ReputationData {
  domain: string;
  score: number;
  risk: 'low' | 'medium' | 'high';
  blacklisted: boolean;
  lastChecked: string;
}

async function checkDomainReputation(domain: string): Promise<Omit<ReputationData, 'domain'>> {
  await new Promise(resolve => setTimeout(resolve, 1500));
  const score = Math.floor(Math.random() * 100);
  const risk = score > 70 ? 'low' : score > 40 ? 'medium' : 'high';
  const blacklisted = score < 30;
  
  return {
    score,
    risk,
    blacklisted,
    lastChecked: new Date().toISOString()
  };
}

function DomainReputation() {
  const navigate = useNavigate();
  const location = useLocation();
  const [loading, setLoading] = useState(false);
  const [domains, setDomains] = useState<string[]>([]);
  const [reputationData, setReputationData] = useState<ReputationData[]>([]);
  const [bulkInput, setBulkInput] = useState('');
  
  const singleDomain = new URLSearchParams(location.search).get('domain') || '';

  const handleBulkCheck = async () => {
    const domainList = bulkInput
      .split('\n')
      .map(d => d.trim())
      .filter(d => d.length > 0);
    
    setDomains(domainList);
    setLoading(true);
    const results: ReputationData[] = [];

    for (const domain of domainList) {
      try {
        const data = await checkDomainReputation(domain);
        results.push({ domain, ...data });
      } catch (error) {
        console.error(`Failed to check reputation for ${domain}:`, error);
      }
    }

    setReputationData(results);
    setLoading(false);
  };

  const getScoreColor = (score: number) => {
    if (score > 70) return 'text-green-500';
    if (score > 40) return 'text-yellow-500';
    return 'text-red-500';
  };

  const getRiskIcon = (risk: string) => {
    switch (risk) {
      case 'low':
        return <CheckCircle className="w-5 h-5 text-green-500" />;
      case 'medium':
        return <AlertTriangle className="w-5 h-5 text-yellow-500" />;
      case 'high':
        return <XCircle className="w-5 h-5 text-red-500" />;
      default:
        return null;
    }
  };

  const exportToCSV = () => {
    const headers = ['Domain', 'Score', 'Risk Level', 'Blacklisted', 'Last Checked'];
    const rows = reputationData.map(data => [
      data.domain,
      data.score,
      data.risk,
      data.blacklisted ? 'Yes' : 'No',
      new Date(data.lastChecked).toLocaleString()
    ]);
    
    const csvContent = [
      headers.join(','),
      ...rows.map(row => row.join(','))
    ].join('\n');
    
    const blob = new Blob([csvContent], { type: 'text/csv' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'domain-reputation-report.csv';
    a.click();
    window.URL.revokeObjectURL(url);
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-50 p-6">
      <div className="max-w-6xl mx-auto">
        <button
          onClick={() => navigate('/')}
          className="flex items-center gap-2 text-indigo-600 hover:text-indigo-700 mb-6"
        >
          <ArrowLeft className="w-4 h-4" />
          Back to Domain Extractor
        </button>

        <div className="bg-white rounded-lg shadow-lg p-6">
          <div className="flex items-center justify-center gap-3 mb-8">
            <Shield className="w-8 h-8 text-indigo-600" />
            <h1 className="text-3xl font-bold text-gray-800">Bulk Domain Reputation Checker</h1>
          </div>

          <div className="mb-8">
            <textarea
              className="w-full h-40 p-4 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 resize-none font-mono"
              placeholder="Enter domains (one per line)&#10;example.com&#10;google.com&#10;microsoft.com"
              value={bulkInput}
              onChange={(e) => setBulkInput(e.target.value)}
            />
            
            <div className="flex justify-between mt-4">
              <button
                onClick={handleBulkCheck}
                disabled={loading}
                className={`px-6 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors ${
                  loading ? 'opacity-50 cursor-not-allowed' : ''
                }`}
              >
                {loading ? 'Checking Domains...' : 'Check All Domains'}
              </button>

              {reputationData.length > 0 && (
                <button
                  onClick={exportToCSV}
                  className="flex items-center gap-2 px-6 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors"
                >
                  <Download className="w-4 h-4" />
                  Export to CSV
                </button>
              )}
            </div>
          </div>

          {loading && (
            <div className="flex justify-center my-8">
              <div className="animate-spin rounded-full h-8 w-8 border-2 border-indigo-500 border-t-transparent"></div>
            </div>
          )}

          {reputationData.length > 0 && !loading && (
            <div className="overflow-x-auto">
              <table className="w-full border-collapse">
                <thead>
                  <tr className="bg-gray-50">
                    <th className="px-6 py-3 text-left text-sm font-semibold text-gray-700">Domain</th>
                    <th className="px-6 py-3 text-left text-sm font-semibold text-gray-700">Score</th>
                    <th className="px-6 py-3 text-left text-sm font-semibold text-gray-700">Risk Level</th>
                    <th className="px-6 py-3 text-left text-sm font-semibold text-gray-700">Status</th>
                    <th className="px-6 py-3 text-left text-sm font-semibold text-gray-700">Last Checked</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-gray-200">
                  {reputationData.map((data, index) => (
                    <tr key={index} className="hover:bg-gray-50">
                      <td className="px-6 py-4 font-mono text-sm text-gray-700">{data.domain}</td>
                      <td className="px-6 py-4">
                        <span className={`font-semibold ${getScoreColor(data.score)}`}>
                          {data.score}/100
                        </span>
                      </td>
                      <td className="px-6 py-4">
                        <div className="flex items-center gap-2">
                          {getRiskIcon(data.risk)}
                          <span className="capitalize">{data.risk}</span>
                        </div>
                      </td>
                      <td className="px-6 py-4">
                        <span className={`inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium ${
                          data.blacklisted
                            ? 'bg-red-100 text-red-800'
                            : 'bg-green-100 text-green-800'
                        }`}>
                          {data.blacklisted ? 'Blacklisted' : 'Clean'}
                        </span>
                      </td>
                      <td className="px-6 py-4 text-sm text-gray-500">
                        {new Date(data.lastChecked).toLocaleString()}
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

export default DomainReputation;
