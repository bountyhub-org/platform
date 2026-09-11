/**
 * Validates a bounty submission URL, supporting both upstream PRs and verified fork comparison links
 * when repository policies restrict external PR creation.
 */
export function validateSubmissionTarget(url: string, repoPolicy: 'open' | 'collaborators_only'): boolean {
  const upstreamPrRegex = /^https:\/\/github\.com\/[\w-]+\/[\w-]+\/pull\/\d+$/;
  const forkCompareRegex = /^https:\/\/github\.com\/[\w-]+\/[\w-]+\/compare\/[\w-\.\/]+...[\w-:]+[\w-\.\/]+(?:\?expand=1)?$/;

  if (upstreamPrRegex.test(url)) {
    return true;
  }

  if (repoPolicy === 'collaborators_only' && forkCompareRegex.test(url)) {
    // Allow verified fork compare URLs when upstream blocks external PRs
    return true;
  }

  return false;
}