# CS5335HW04
Matthew Pashkowsky
Youtube Kerbin Video: 
My strategy is a greedy strategy. Overall, the ship does not plan a full trip at once, but picks an individual moon it is going to go to next. In order to conserve fuel and simplify the option space, the ship only analyzes Hohmann Transfers. The ship analyzes the inner three most moons it has not been to yet, and checks if it can do a Hohmann Transfer almost immediately on either the second or third moon. If one isn't avaiable, it will go to the innermost moon. Once it intercepts the target moon, it recircularizes its orbit around the central planet, and then looks again at where it is going to go next. The goal is to minimize time on the overall journey by minimizing every step's time.

Code: 
CLEARSCREEN.
STAGE.
Local Normal is “NORMAL”.
Local AntiNormal is “ANTINORMAL”.
Local Prograde is “PROGRADE”.
Local Retrograde is “RETROGRADE”.
Local Planet is ship:body.
Local MoonNumber is ship:body:orbitingchildren:length.
Local MoonsToVisitNumber is MoonNumber.
Local MoonsToVisit is list().
Local MoonsOrbitVelocity is list().
Local MoonsBurnTimes is list().
Local ShipVelocity is Ship:Velocity:Orbit:Mag.
Local Var is 0.
Until Var = MoonNumber {
	MoonsToVisit:Add(ship:body:orbitingchildren[Var]).
	MoonsOrbitVelocity:Add(360/MoonsToVisit[Var]:Orbit:Period*MoonsToVisit[Var]:Orbit:Apoapsis). 
	Set Var to Var + 1.
}
Function ThetaPos {
	Parameter Position.
	Set a0 to ARCTAN2(Position:z, Position:x).
	Local a1 is mod(a0 + 360, 360).
	Return a1.
}.
Until MoonsToVisitNumber = 0 {
Local TBetweenTargets is 0.
Local ShipPosition is ThetaPos(Ship:Position - Planet:Position).
Local Target is Planet.
Local TargetNumber is 0.

Local MoonsTargeting is list().
If MoonsToVisit:Length < 3 {
	Set MoonsTargeting to MoonsToVisit.
} Else {
	Set MoonsTargeting to list(MoonsToVisit[0],MoonsToVisit[1],MoonsToVisit[2]).
}
Set Var to 1.
Until Var = MoonsTargeting:Length {
Local MoonPosition is ThetaPos(MoonsTargeting[Var]:Position - Planet:Position).
Set TBetweenOrbits to ((ABS(Ship:Apoapsis - MoonsTargeting[Var]:Apoapsis))/((ShipVelocity + MoonsOrbitVelocity[Var])/2)).
If mod(MoonPosition + MoonsOrbitVelocity[Var]*TBetweenOrbits - ShipPosition + 360,360) < 10 {
Set Target to MoonsTargeting[Var].
Set TargetNumber to Var.
Set Var to MoonTargeting:Length.
} Else {
Set Var to Var + 1.
}
}
If TargetNumber = 0 {
Set Target to MoonsTargeting[0].
}
Print Target.
Local NormalNeeded is 0.
Local NormalValue is 0.
If Abs(Ship:Orbit:Inclination - Target:Orbit:Inclination) > 0.1 {
Set NormalNeeded to 1.
If Ship:Orbit:Inclination < Target:Orbit:Inclination {
Set NormalValue to NormalValue + 1.
}
If mod(ThetaPos(Target:Position) - ShipPosition + 360,360) > 180 {
Set NormalValue to NormalValue + 1.
}
}
If NormalNeeded = 1 {
If mod(NormalValue,2) = 1 {
SAS On.
Set SASMode To AntiNormal.
Until Abs(Ship:Orbit:Inclination - Target:Orbit:Inclination) < 0.1 {
	Set Throttle to 1.0.
}
Set Throttle to 0.0.
SAS Off.
} Else {
SAS On.
Set SASMode To Normal.
Until Abs(Ship:Orbit:Inclination - Target:Orbit:Inclination) < 0.1 {
	Set Throttle to 1.0.
}
Set Throttle to 0.0.
SAS Off.
}
}
Local BurnPosition is mod(ThetaPos(Target:Position-Planet:Position) + MoonsOrbitVelocity[TargetNumber]*TBetweenOrbits*360/Target:Orbit:Apoapsis - ShipPosition + 450-Ship:Orbit:Inclination*15,360).
Print BurnPosition.
Print ShipVelocity.
Local TimeToBurn is 0.98*BurnPosition*Ship:Orbit:Period/360.
Print TimeToBurn.
Wait TimeToBurn.
Until Ship:Apoapsis > Target:Apoapsis + 1000{
SAS On.
Set SASMode To Prograde.
Set Throttle to 1.0.
}
Until Ship:Body = Target {
Set Throttle to 0.0.
SAS Off.
}
Until Ship:Body = Planet {
Set Throttle to 0.0.
SAS Off.
}
Until Ship:Apoapsis - Ship:Altitude < 1000 {
Set Throttle to 0.0.
SAS Off.
}
Until Ship:Periapsis > 1.1*Target:Apoapsis {
SAS On.
Set SASMode To Prograde.
Set Throttle to 1.0.
}
Set Throttle to 0.0.
MoonsToVisit:Remove(TargetNumber).
MoonsOrbitVelocity:Remove(TargetNumber).
Set MoonsToVisitNumber to MoonsToVisitNumber - 1.
}
