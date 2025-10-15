<div>
  <div style="display:inline-block; vertical-align:top; margin-right:2em;">
    <h3 style="margin:0;">Theory</h3>
    <ul style="margin:0; padding-left:0; list-style:none;">
      <li><a href="./theory/1_Understanding_Microservices.html">Understanding Microservices</a></li>
      <li><a href="./theory/2_Key_Ideas_Behind_Microservices.html">Key Ideas Behind Microservices</a></li>
      <li><a href="./theory/3_Aligning_Architecture_and_Team_Organization.html">Aligning Architecture and Team Organization</a></li>
      <li><a href="./theory/4_The_Monolith.html">The Monolith</a></li>
      <li><a href="./theory/5_Why_Microservices_Are_Useful.html">Why Microservices Are Useful</a></li>
      <!-- Haselem Microservice Pain Points 52-->
    </ul>   
    <h3 style="margin:0;">Design Patterns</h3>
    <ul style="margin:0; padding-left:0px; list-style:none;">
      <li><a href="./DesignPatterns/1_Service_Discovery">Service Discovery</a></li>
    </ul>
  </div>

  <div style="display:inline-block; vertical-align:top;">
    <h3 style="margin:0;">Resilience4j</h3>
    <ul style="margin:0; padding-left:0px; list-style:none;">
      <li><a href="./Resilience4j/Retry_with_Resilience4j.html">Retry</a></li>
    </ul>
    <h3 style="margin:0;">Microservices with Spring Boot</h3>
    <ul style="margin:0; padding-left:0px; list-style:none;">
    <li><a href="./MicroservicesWithSpringBoot/1_The_Rise_of_Microservices">The Rise of Microservices</a></li>
    <!-- HASELEM Challenges with microservices EJ 43-->
    </ul>
  </div>
</div>


<!-- 
### Resilience4j
- [Retry](.md)
- Rate Limiting 
- Timeouts 
- Bulkhead 
- Circuit Breaker
- Retry with Spring Boot
- Rate Limiting with Spring Boot
- Timeouts with Spring Boot -->

<!-- https://reflectoring.io/rate-limiting-with-resilience4j/ -->
<!-- https://reflectoring.io/time-limiting-with-resilience4j/ -->
<!-- https://reflectoring.io/bulkhead-with-resilience4j/ -->
<!-- https://reflectoring.io/circuitbreaker-with-resilience4j/ -->
<!-- https://reflectoring.io/retry-with-springboot-resilience4j/ -->
<!-- https://reflectoring.io/rate-limiting-with-springboot-resilience4j/ -->
<!-- https://reflectoring.io/time-limiting-with-springboot-resilience4j/ -->

---

[Home](./../README.md)


<!-- @startuml
!define RECTANGLE class

skinparam rectangle {
  BackgroundColor #DDEEFF
  BorderColor #3333AA
}

title Learning Platform Microservices (Async + Sync)

RECTANGLE "User Service" as User {
  - Manage profiles
  - Track progress
}

RECTANGLE "Course Service" as Course {
  - Store courses & lessons
  - Provide content APIs
}

RECTANGLE "Exercise Service" as Exercise {
  - Serve exercises
  - Validate answers
}

RECTANGLE "Scoring Service" as Scoring {
  - Calculate points
  - Manage leaderboard
}

RECTANGLE "Recommendation Service" as Recommendation {
  - Suggest next lessons
  - Analyze user performance
}

RECTANGLE "Notification Service" as Notification {
  - Send email/push notifications
}

RECTANGLE "Subscription Service" as Subscription {
  - Manage payments & plans
}

'-----------------------------
' Synchronous API calls (solid arrows)
'-----------------------------
User --> Course : Request courses
User --> Exercise : Request exercises
Subscription --> Exercise : Check access
Subscription --> Course : Check access

'-----------------------------
' Asynchronous events (dashed arrows)
'-----------------------------
User ..> Recommendation : UserProgressUpdated [Kafka Topic]
Exercise ..> Scoring : AnswerSubmitted [Kafka Topic]
Scoring ..> Notification : ScoreUpdate [Kafka Topic]
Recommendation ..> Notification : RecommendationReady [Kafka Topic]
Course ..> Recommendation : CourseCreated [Kafka Topic]

@enduml -->
