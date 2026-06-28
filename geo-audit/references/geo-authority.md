# GEO: Authority & Trust Signals

### Author Credentials
**Grep**: `author.*bio|credentials|jobTitle|expertise`
**Check**: Professional credentials (Chef, Recipe Developer) | Specialties/expertise | Years experience | Training/education | Publications/awards/media

**Impact**: High - AI heavily weights expert authorship

```tsx
<AuthorCard>
  <h3>{author.name}</h3>
  <p className="credentials">Professional Chef | 15yrs | CIA</p>
  <p className="expertise">Specializes: {author.specialties.join(", ")}</p>
  <p>{author.bio}</p>
</AuthorCard>
```

### Reviews & Ratings
**Grep**: `review|rating|stars|aggregateRating`
**Check**: Star system (1-5) | Review count | Review text | Verified reviewers | Helpful votes | Author responses
**Impact**: Critical - primary trust signal

**Schema.org**:
```typescript
{
  "@type": "Recipe",
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": 4.7,
    "reviewCount": 234,
    "bestRating": 5,
    "worstRating": 1,
  },
  "review": [
    {
      "@type": "Review",
      "author": { "@type": "Person", "name": "Jane Smith" },
      "reviewRating": { "@type": "Rating", "ratingValue": 5 },
      "reviewBody": "Delicious! Made exactly as written...",
      "datePublished": "2026-05-15",
    }
  ]
}
```

### Recipe Testing
**Grep**: `tested|test.*kitchen|verified|triple.*test`
**Check**: "Tested X times" | "Test kitchen approved" | "Community tested" (fork counts) | Testing notes/variations
**Why**: AI prefers validated recipes

```tsx
<QualityBadge>
  <Icon name="check-circle" />
  <span>Tested 3x in kitchen</span>
</QualityBadge>
```

### Community Engagement
**Grep**: `comment|fork|save|bookmark|favorite`
**Check**: Comment count | Fork/remix count | Save/bookmark count | View count | Share count

```tsx
<Engagement>
  <span>{recipe.forkCount} remixes</span>
  <span>{recipe.saveCount} saves</span>
  <span>{recipe.commentCount} comments</span>
</Engagement>
```
**Why**: High engagement → popular, trusted

### Version History
**Grep**: `version|history|updated|revised`
**Check**: Update history visible | Changelog (what improved, why) | Versions/dates | "Updated per feedback"
**Impact**: Med-High - shows iterative improvement

### External Links
**Grep**: `href.*http` (exclude nav)
**Check**: Links to authoritative sources (USDA, serious food sites, cookbooks) | Technique refs | Ingredient sourcing | Minimal spammy affiliates
**Issue**: Too many affiliate links → reduced trust

### Professional Photography
**Check**: High-quality photos (not stock) | Multiple angles (process + final) | Consistent style | Proper attribution

### Editorial Standards
Quality: Grammar/spelling | Consistent formatting | Standard structure | Precise measurements | Realistic times
**Issue**: Sloppy editing → reduced authority
**Impact**: Medium - AI may deprioritize

## Common Issues

### Anonymous Authors
No profile or minimal info
**Impact**: Critical - lacks credibility
**Fix**: Require bio, credentials on all recipes

### No Reviews/Ratings
No user feedback
**Impact**: High - can't demonstrate validation
**Fix**: Implement rating/review system

### Hidden Engagement
Metrics exist but not displayed
**Impact**: Medium - missed popularity signal
**Fix**: Surface metrics on pages

### Stale Content
Published years ago, never updated
**Impact**: Low-Med - seems outdated
**Fix**: Add "last updated", encourage updates

### No Testing Indicators
Unclear if tested
**Impact**: Medium - uncertain reliability
**Fix**: Testing badges, notes

### Broken External Links
Citations → 404s
**Impact**: Medium - reduces credibility
**Fix**: Link health checks, update/remove broken

## Fixes

### Enhanced Author Profiles
**Schema**:
```sql
ALTER TABLE users
ADD credentials TEXT,
ADD specialties TEXT[],
ADD years_experience INT,
ADD education TEXT,
ADD awards TEXT[],
ADD media_appearances TEXT[];
```

**Display**:
```tsx
<AuthorProfile>
  <h3>{author.name}</h3>
  {author.credentials && <p>{author.credentials}</p>}
  {author.specialties && <p>Specialties: {author.specialties.join(", ")}</p>}
  {author.yearsExperience && <p>{author.yearsExperience}yrs experience</p>}
  <p>{author.bio}</p>
  <RecipeCount>{author.recipeCount} recipes | {author.totalForks} remixes</RecipeCount>
</AuthorProfile>
```

### Testing Indicator
**DB**:
```sql
ALTER TABLE recipes ADD times_tested INT DEFAULT 0, ADD test_notes TEXT;
```

**Display**:
```tsx
{recipe.timesTested > 0 && (
  <QualityBadge>
    <CheckCircle />
    <span>Tested {recipe.timesTested}x</span>
  </QualityBadge>
)}
```

### Surface Engagement
```tsx
<Metrics>
  <Metric icon={<Eye />} label="Views" value={recipe.viewCount} />
  <Metric icon={<GitBranch />} label="Remixes" value={recipe.forkCount} />
  <Metric icon={<Bookmark />} label="Saves" value={recipe.saveCount} />
  <Metric icon={<MessageCircle />} label="Comments" value={recipe.commentCount} />
</Metrics>
```

### Version History
**DB**:
```sql
CREATE TABLE recipe_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipe_id UUID REFERENCES recipes(id) ON DELETE CASCADE,
  version_number INT NOT NULL,
  changes TEXT NOT NULL,
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  updated_by UUID REFERENCES users(id)
);
```

### Review System Schema.org
```typescript
{
  "@type": "Recipe",
  "aggregateRating": recipe.reviewCount > 0 ? {
    "@type": "AggregateRating",
    "ratingValue": recipe.averageRating,
    "reviewCount": recipe.reviewCount,
    "bestRating": 5,
  } : undefined,
  "review": recipe.reviews.map(r => ({
    "@type": "Review",
    "author": { "@type": "Person", "name": r.author.name, "image": r.author.avatar },
    "reviewRating": { "@type": "Rating", "ratingValue": r.rating, "bestRating": 5 },
    "reviewBody": r.comment,
    "datePublished": r.createdAt,
  }))
}
```

### Trust Badges
```tsx
export function TrustBadges({ recipe }) {
  return (
    <div className="trust-badges">
      {recipe.timesTested >= 3 && <Badge><CheckCircle /> Kitchen Tested</Badge>}
      {recipe.averageRating >= 4.5 && recipe.reviewCount >= 10 && <Badge><Star /> Highly Rated</Badge>}
      {recipe.forkCount >= 50 && <Badge><GitBranch /> Community Favorite</Badge>}
      {recipe.author.credentials && <Badge><Award /> Expert Recipe</Badge>}
    </div>
  )
}
```

## Schema.org Full Authority
```typescript
{
  "@type": "Recipe",
  "author": {
    "@type": "Person",
    "name": author.name,
    "url": authorUrl,
    "image": author.avatar,
    "jobTitle": author.credentials,
    "knowsAbout": author.specialties,
    "sameAs": author.socialLinks, // Social proof
  },
  "aggregateRating": { /* rating */ },
  "review": [ /* reviews */ ],
  "interactionStatistic": [
    { "@type": "InteractionCounter", "interactionType": "CommentAction", "userInteractionCount": recipe.commentCount },
    { "@type": "InteractionCounter", "interactionType": "ShareAction", "userInteractionCount": recipe.shareCount },
  ],
  "datePublished": recipe.createdAt,
  "dateModified": recipe.updatedAt,
}
```

## Validation

AI test: ChatGPT recommend recipes → check if yours appear w/ author/ratings
Trust audit per recipe: Author w/ credentials? | Rating (≥5 reviews)? | Engagement visible? | Testing? | Professional photo? | Last updated? | Citations?
**Target**: 5+ trust signals/recipe

