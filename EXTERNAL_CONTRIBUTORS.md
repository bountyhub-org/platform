// packages/platform-core/src/claims/validator.ts

export interface ClaimSubmissionInput {
  bountyId: string;
  contributorGitHubUsername: string;
  pullRequestUrl?: string;
  forkComparisonUrl?: string;
  onboardingPrUrl?: string;
}

export async function validateAndProcessClaim(input: ClaimSubmissionInput): Promise<ClaimResult> {
  // 1. Verify if upstream PR URL is provided and valid
  if (input.pullRequestUrl) {
    return await processStandardUpstreamPRClaim(input);
  }

  // 2. Fallback path for repositories blocking external PRs (collaborators_only policy)
  if (input.forkComparisonUrl && input.onboardingPrUrl) {
    const isVerifiedContributor = await verifyExternalContributorRegistry(
      input.contributorGitHubUsername,
      input.onboardingPrUrl
    );

    if (isVerifiedContributor) {
      return await createProxyClaimOrTriggerMaintainerAssignment({
        bountyId: input.bountyId,
        contributorGitHubUsername: input.contributorGitHubUsername,
        forkComparisonUrl: input.forkComparisonUrl,
        status: 'PENDING_MAINTAINER_PROXY_PR'
      });
    }
  }

  throw new Error(
    'Invalid claim: Must provide a valid upstream pull request URL or a verified fork comparison URL accompanied by proof of onboarding.'
  );
}