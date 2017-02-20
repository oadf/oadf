# Open Athletics Data Format

## Abstract

Open Athletics Data Format (OADF) describes a schema of data for athletics competitions as well as a certain set of 
queries to query the data. A consistent way of specifing data makes it possible to easily exchange data between
systems.

This document is currently under development.

## Schema

This document lists all types specified in the OADF schema. It uses the [GraphQL](http://facebook.github.io/graphql) 
type system.

### Age Group

A definition of a group of people starting together.

```
type AgeGroup {
  id: ID!
  name: String!
}
```

### Association

An organization in which multiple clubs or associations are grouped together.

```
type Association {
  id: ID!
  name: String!
  association: Association
}
```

### Athlete

A person participating in the meeting or member of a club.

```
type Athlete {
  id: ID!
  bib: Int
  firstName: String!
  lastName: String!
  yob: Int!
  gender: String!
  citizenship: String
  club: Club
  memberships: [ClubMember]
}
```

#### Descriptions

**bib**: If the context of the query is a meeting the bib of the athlete in this
meeting is displayed.

**citizenship**: The IOC 3-letter code of the country.

**club**: The current club of an athlete. Depending on the context this is either
the club for a meeting which is queried or the club the athlete starts for at the 
time of the query.

**memberships**: A list of all club memberships of an athlete during his career.

### AthleteResult

```
type AthleteResult implements IResult, IAthleteResult {
  id: ID!
  group: Group!
  athlete: Athlete!
  position: Int
  performance: Float
  exception: String
  qualified: String
  wind: Float
  attempts: [Attempt]
  heights: [HeightResult]
  combinedResults: [CombinedResult]
  comment: String
  weight: Float
}
```

#### Descriptions

**combinedResults**: If this result is a result of a combined event a list of
all single results who make up the total result can be received from here.

**weight**: If multiple athletes start together in a group and use different
weights specify the weight that was used by the athlete here.

### Attempt 

```
type Attempt {
  id: ID!
  result: AthleteResult!
  number: Int!
  performance: Performance
  wind: Float
  exception: String
}
```

### Club

A club for which athletes participate in competitions.

```
type Club {
  id: ID!
  name: String!
  number: ID
  association: Association
}
```

### Club Member

Describes a time span in which an athlete started for a certain club.

```
type ClubMember {
  id: ID!
  athlete: Athlete!
  club: Club!
  startDate: String
  endDate: String
  licenseNumber: ID
}
```

### Combined Discipline

A discipline which was part of an combined event.

```
type CombinedDiscipline {
  id: ID!
  event: Event!
  discipline: Discipline!
  order: Int
}
```

### Combined Result

A subresult which was part of a combined event.

```
type CombinedResult implements IResult, IAthleteResult {
  id: ID!
  athlete: Athlete!
  parentResult: AthleteResult!
  performance: Float
  exception: String
  wind: Float
  points: Int
}
```

#### Descriptions

**parentResult:** Reference to the result this result is part of.

**points:** The points received for the performance.

### Discipline

A discipline in which athletes or teams compete.

```
type Discipline {
  id: ID!
  name: String!
}
```

### Event

An event defined by a unique combination of age group and discipline.

```
type Event {
  id: ID!
  meeting: Meeting!
  ageGroup: AgeGroup!
  discipline: Discipline!
  rounds: [Round]
  height: Int
  weight: Int
  combinedDisciplines: [CombinedDiscipline]
}
```

#### Descriptions

**height:** The height of hurdles or others in mm.

**weight:** The weight of javelin, shot put or others in grams.

**combinedDisciplines:** If a combined event, a list of disciplines that
were part of the combined competition.

### Group

A group of athletes starting together in a round (examples Group A, Heat 1)

```
type Group {
  id: ID!
  round: Round!
  number: Int!
  wind: Float
  time: String
  results: [Result]
  splitTimes: [SplitTime]
  comment: String
}
```

#### Descriptions

**time:** The time if there is a more exact time then the round time.

**wind:** Wind for all results in the group.

**heights:** A list of all heights that where put up in the group.

**splitTimes:** A list of split time from the race of this group.

### Height

A height which was put up in an group.

```
type Height {
  id: ID!
  group: Group!
  height: Float!
}
```

### HeightResult 

The performance for an athlete at a specific height.

```
type HeightResult {
  id: ID!,
  height: Height!
  result: AthleteResult!
  performance: HeightPerformance!
}
```

#### Descriptions

**performance:** Performance as a string of X, O and -

### IAthleteResult

Additional fields for athlete results used in standard and combined results.

```
interface IAthleteResult {
  athlete: Athlete!
  wind: Float
  attempts: [Attempt]
  heights: [HeightResult]
}
```

#### Descriptions

**wind**: If the wind only refers to this single result it is displayed here.
Else it can be found at a higher level.

### IResult

Interface for different types of results. This can be an athlete, a combined or
team result.

```
interface IResult {
  id: ID!
  group: Group!
  position: Int
  performance: Performance
  exception: String
  automaticTiming: Boolean
  qualified: String
  comment: String
}
```

#### Descriptions

**position**: The position within the group.

**performance**: The performance as a number. Minutes will also be converted to seconds.

**exception**: Exception type if attempt was not completed successfully.

**qualified**: If the starter qualified for the next round or waived his right 
to go to the next round.

### Meeting

```
type Meeting {
  id: ID!
  name: String!
  events: [Event]
  venue: Venue
  startDate: String!
  endDate: String
}
```

### Round

A round for an event (examples: Qualification, Heats, Final).

```
type Round {
  id: ID!
  event: Event!
  name: String!
  date: String
  time: String
  comment: String
  combinedDiscipline: CombinedDiscipline
  groups: [Group]
}
```

#### Descriptions

**combinedDiscipline**: If this is a round within a combined event the discipline
of the round is referenced here.

### SplitTime

Split times taken during a race of a group.

```
type SplitTime {
  id: ID!
  group: Group!
  distance: Int!
  time: Float!
  athlete: Athlete
}
```

####Descriptions

**distance:** The distance that was run as this split time was taken in meters.

**time:** The split time that was taken in seconds.

**athlete:** The athlete that was first at the split time.

### TeamMember

```
type TeamMember {
  id: ID!
  result: TeamResult!
  athlete: Athlete!
  performance: Float
}
```

### TeamResult

```
type TeamResult implements IResult {
  id: ID!
  group: Group!
  position: Int
  performance: Performance
  exception: String
  qualified: String
  club: Club!
  country: String
  teamMembers: [TeamMember]
  comment: String
}
```

### Venue

A venue where an event or the whole meeting takes place

```
type Venue {
  id: ID!
  name: String!
  city: String
}
```

## About

This specification is a joined work from several vendors who create software for athletics competitions in Germany.
We are looking for other athletics software vendors worldwide to join our effort to resonate data sharing for athletics competitions. 

The data format which is specified here is or will be supported by the following:

- [Leichtathletik-Datenbank](http://www.leichtathletik-datenbank.de)

## Editors

Christoph Kraemer, Leichtathletik-Datenbank
