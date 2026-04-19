---
title: 一个简单的Umat子程序
publishDate: 2025-06-15 07:32:27
description: '笔记 1'
tags:
  - UMAT
  - 基础
  - YSU
language: '中文'
---



# Power Law UMAT 

December 19, 1995 

## 1 Mechanics 

No summation in the following expressions!!  

$$
\sigma _ { i j } = S _ { i j } + p \delta _ { i j }
$$

$$
p = k \ t r \ \epsilon
$$

For shear components introduce engineering strain:  

$$
\gamma _ { i j } = 2 \epsilon _ { i j }
$$

$$
S _ { i i } = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \sigma _ { 0 } \frac { \epsilon _ { i i } } { \epsilon _ { 0 } }  \mathrm { a n d }  S _ { i j } = \frac { 1 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \sigma _ { 0 } \frac { \gamma _ { i j } } { \epsilon _ { 0 } }
$$

$$
\begin{array}{rl} { \overline { { \epsilon } } } & { = \left( \frac { 2 } { 3 } \epsilon _ { i j } \epsilon _ { i j } \right) ^{\frac { 1} { 2 } } = \left( \frac { 2 } { 3 } \left( \sum \epsilon _ { i i } ^{2} + 2 \sum \epsilon _ { i j } ^{2} \right) \right) ^{\frac { 1} { 2 } } } \\& { = \left( \frac { 2 } { 3 } \left( \sum \epsilon _ { i i } ^{2} + \frac { 1 } { 2 } \sum \gamma _ { i j } ^{2} \right) \right) ^{\frac { 1} { 2 } } } \end{array}
$$

$$
\begin{array}{r} { \frac { \partial \epsilon _ { i i } } { \partial \epsilon _ { j j } } = \pmb { \delta } _ { i j }  \mathrm { a n d }  \frac { \partial \gamma _ { i j } } { \partial \gamma _ { k l } } = \pmb { \delta } _ { i k } \pmb { \delta } _ { j l } } \end{array}
$$

$$
I _ { i j k l } = \frac { 1 } { 2 } ( \delta _ { i k } \delta _ { j l } + \delta _ { i l } \delta _ { j k } )
$$

$$
\boldsymbol { J } = \frac { \partial \Delta \sigma _ { i j } ( t + \Delta t ) } { \partial \Delta \epsilon _ { k l } ( t + \Delta t ) } = \frac { \partial \sigma _ { i j } ( t + \Delta t ) } { \partial \epsilon _ { k l } ( t + \Delta t ) }
$$

$$
\frac { \partial \, t r \, \epsilon } { \partial \epsilon _ { k l } } = \delta _ { k l }
$$

$$
\frac { \partial \overline { { \epsilon } } } { \partial \epsilon _ { k k } } = \frac { 1 } { 2 \overline { { \epsilon } } } \frac { \partial \overline { { \epsilon } } ^{2} } { \partial \epsilon _ { k k } } = \frac { 1 } { 2 \overline { { \epsilon } } } \frac { 2 } { 3 } \frac { \partial \left( \sum \epsilon _ { i i } ^{2} \right) } { \partial \epsilon _ { k k } } = \frac { 2 } { 3 \overline { { \epsilon } } } \epsilon _ { k k }
$$

$$
\frac { \partial \overline { { \epsilon } } } { \partial \gamma _ { k l } } = \frac { 1 } { 2 \overline { { \epsilon } } } \frac { \partial \overline { { \epsilon } } ^{2} } { \partial \gamma _ { k l } } = \frac { 1 } { 2 \overline { { \epsilon } } } \frac { 2 } { 3 } \frac { \partial \left( \frac { 1 } { 2 } \sum \gamma _ { i j } ^{2} \right) } { \partial \gamma _ { k l } } = \frac { 1 } { 3 \overline { { \epsilon } } } \gamma _ { k l }
$$

Now we can compute $\frac{\partial S}{\partial \epsilon}$:  

$$
\begin{array}{rl} { \frac { \partial S _ { i i } } { \partial \epsilon _ { k k } } } & { = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { m - 1 } { \overline { { \epsilon } } } \frac { \partial \overline { { \epsilon } } } { \partial \epsilon _ { k k } } \epsilon _ { i i } + \frac { \partial \epsilon _ { i i } } { \partial \epsilon _ { k k } } \right) } \\& { = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { 2 ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \epsilon _ { k k } \epsilon _ { i i } + \delta _ { i k } \right) } \end{array}
$$

$$
\begin{array}{rl} { \frac { \partial S _ { i j } } { \partial \epsilon _ { k k } } } & { = \frac { 1 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { m - 1 } { \tau } \frac { \partial \overline { { \epsilon } } } { \partial \epsilon _ { k k } } \gamma _ { i j } + \frac { \partial \gamma _ { i j } } { \partial \epsilon _ { k k } } \right) } \\& { = \frac { 1 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \frac { 2 ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \epsilon _ { k k } \gamma _ { i j } } \end{array}
$$

$$
\begin{array}{rl} { \frac { \partial S _ { i i } } { \partial \gamma _ { k l } } } & { = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { m - 1 } { \overline { { \epsilon } } } \frac { \partial \overline { { \epsilon } } } { \partial \gamma _ { k l } } \epsilon _ { i i } + \frac { \partial \epsilon _ { i i } } { \partial \gamma _ { k l } } \right) } \\& { = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \frac { ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \gamma _ { k l } \epsilon _ { i i } } \end{array}
$$

$$
\begin{array}{rl} { \frac { \partial S _ { i j } } { \partial \gamma _ { k l } } } & { = \frac { 1 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { m - 1 } { \overline { { \epsilon } } } \frac { \partial \overline { { \epsilon } } } { \partial \gamma _ { k l } } \boldsymbol { \gamma } _ { i j } + \frac { \partial \boldsymbol { \gamma } _ { i j } } { \partial \boldsymbol { \gamma } _ { k l } } \right) } \\& { = \frac { 1 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \boldsymbol { \epsilon } _ { k l } \boldsymbol { \epsilon } _ { i j } + \boldsymbol { \delta } _ { i k } \boldsymbol { \delta } _ { j l } \right) } \\& { \frac { \partial p } { \partial \epsilon _ { k k } } \boldsymbol { \delta } _ { i j } = k \frac { \partial \, t r \, \boldsymbol { \epsilon } } { \partial \epsilon _ { k k } } \boldsymbol { \delta } _ { i j } = k \boldsymbol { \delta } _ { i j }  \mathrm { a n d }  \frac { \partial p } { \partial \gamma _ { k l } } \boldsymbol { \delta } _ { i j } = k \frac { \partial \, t r \, \boldsymbol { \epsilon } } { \partial \gamma _ { k l } } \boldsymbol { \delta } _ { i j } = 0 } \end{array}
$$

Now we can compute Jacobian:  

$$
\begin{array}{rl} { \pmb { J } _ { i i k k } } & { = \frac { \partial \pmb { \sigma } _ { i i } } { \partial \epsilon _ { k k } } = \frac { \partial S _ { i i } } { \partial \epsilon _ { k k } } + \frac { \partial p } { \partial \epsilon _ { k k } } } \\& { = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { 2 ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \pmb { \epsilon } _ { k k } \pmb { \epsilon } _ { i i } + \pmb { \delta } _ { i k } \right) + k } \\{ \pmb { J } _ { i j k k } } & { = \frac { \partial \pmb { \sigma } _ { i j } } { \partial \epsilon _ { k k } } = \frac { \partial S _ { i j } } { \partial \epsilon _ { k k } } } \\& { = \frac { 1 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \frac { 2 ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \pmb { \epsilon } _ { k k } \pmb { \gamma } _ { i j } } \\{ \pmb { J } _ { i i k l } } & { = \frac { \partial \pmb { \sigma } _ { i i } } { \partial \pmb { \gamma } _ { k l } } = \frac { \partial S _ { i i } } { \partial \pmb { \gamma } _ { k l } } + \frac { \partial p } { \partial \pmb { \gamma } _ { k l } } } \\& { = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \frac { ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \pmb { \gamma } _ { k l } \pmb { \epsilon } _ { i i } } \\{ \pmb { J } _ { i j k l } } & { = \frac { \partial \pmb { \sigma } _ { i j } } { \partial \pmb { \gamma } _ { k l } } = \frac { \partial S _ { i j } } { \partial \pmb { \epsilon } _ { k l } } } \\& { = \frac { 1 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \left( \frac { ( m - 1 ) } { 3 \overline { { \epsilon } } ^{2} } \pmb { \epsilon } _ { k l } \pmb { \epsilon } _ { i j } + \pmb { \delta } _ { i k } \pmb { \delta } _ { j l } \right) } \end{array}
$$

and Stress:  

$$
\pmb { \sigma } _ { i i } = \pmb { S } _ { i i } + p = \frac { 2 } { 3 } \left( \frac { \overline { { \epsilon } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \pmb { \epsilon } _ { i i } + k \, t r \, \pmb { \epsilon }
$$

$$
\sigma _ { i j } = S _ { i j } = \frac { 1 } { 3 } \left( \frac { \overline { { c } } } { \epsilon _ { 0 } } \right) ^{m - 1} \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } \gamma _ { i j }
$$



## 2 Coding  

$$
P N L T \ = \ k
$$

$$
T E R M 1 \ = \ { \frac { 2 } { 3 } } \left( { \frac { \overline { { \epsilon } } } { \overline { { \epsilon _ { 0 } } } } } \right) ^{m - 1} { \frac { \sigma _ { 0 } } { \epsilon _ { 0 } } }
$$

$$
T E R M 2 \ = \ { \frac { 2 ( m - 1 ) } { 3 \overline { { c } } ^{2} } }
$$

$$
T E R M 3 \ = \ { \frac { 1 } { 2 } } T E R M 1 \, T E R M 2
$$

For Jacobian:  

$$
\begin{array}{rl} { J _ { i i k k } } & { { } = \ T E R M 1 \left( T E R M 2 \, \epsilon _ { k k } \epsilon _ { i i } + \pmb { \delta } _ { i k } \right) + P N L T } \end{array}
$$

$$
\begin{array}{rl} { J _ { i i k l } } & { { } = \textbf { J } _ { k l i i } = T E R M 3 \, \epsilon _ { i i } \gamma _ { k l } } \end{array}
$$

$$
\begin{array}{rl} { J _ { i j k l } } & { = \frac { 1 } { 2 } T E R M 1 \left( \frac { 1 } { 2 } T E R M 2 \, \gamma _ { i j } \gamma _ { k l } + \delta _ { i k } \delta _ { j l } \right) } \end{array}
$$

For Stress:  

$$
\pmb { \sigma } _ { i i } \ = \ T E R M 1 \, \epsilon _ { i i } + P N L T \, t r \, \epsilon
$$

$$
\sigma _ { i j } \ = \ \frac { 1 } { 2 } T E R M 1 \ \epsilon _ { i j }
$$

## 3 ABAQUS input file: Uniaxial Tension  

```fortran
*LEADING

UMAT - POWER LAW INCOMPRESSIBLE MATERIAL, C3D8 ---- UMATPLT3

*WAVEFRONT MINIMIZATION, SUPPRESS

*NODE,NSET=ALLN

1,0..0.,0.

2,1..0.,0.

3,1..1.,0.

4,0..1.,0.

5,0..0.,1.

6,1..0.,1.

7,1..1.,1.

8,0..1.,1.

*ELEMENT,TYPE=C3D8,ELSET=ALLE

1,1,2,3,4,5,6,7,8

*SOLID SECTION,ELSESET=ALLE,MATERIAL=ALLE

*MATERIAL,NAME=ALLE

*USER MATERIAL,CONSTANTS=7

**E v POWER sig0 eps0 St Tol Pnlt

200.E3,.3,.5,1..,1..,1.E-6,1.E6

*USER SUBROUTINE

SUBROUTINE UMAT(STRESS,STATEV,DDSDDE,SSE,SPD,SCD,

1 RPL,DDSDDT,DRPLDE,DRPLDT,STRAN,DTRAN,

2 TIME,DTIME,TEMP,DTEMP,PREDEF,DPRED,MATERL,NDI,NSHR,NTENS,

3 NSTATV,PROPNS,COORDS,DROT,PNEWD,CELENT,

4 DFGRD0,DFGRD1,NOEL,NPT,KSLAY,KSPT,KSTEP,KINC)

C

INCLUDE 'ABA_PARAM.INC'

C

CHARACTER*8 CMNAME

DIMENSION STRESS(NTENS), STATEV(NSTATV),

1 DDSDDE(NTENS,NTENS),DDSDDT(NTENS),DRPLDE(NTENS),

2 STRAN(NTENS),DSTRAN(NTENS),TIME(2),PREDEF(1),DPRED(1),

3 PROPS(NPROPS),COORDS(3),DROT(3,3),

4 DFGRD0(3,3),DFGRD1(3,3)

C

DIMENSION STRANT(6),DELTA(3,3)

C

C  

PARAMETER (ONE=1.0DO,TWO=2.0DO,THREE=3.0DO,SIX=6.0DO,
1 HALF=.5DO,ZERO = 0.0DO)
DATA NEWTON,TOLER/10,1.D-6/

C
C KRONECKER'S DELTA

C
DATA DELTA /1.0DO,0.0DO,0.0DO,

1 0.0DO,1.0DO,0.0DO,

2 0.0DO,0.0DO,1.0D0/

C
C
UMAT FOR ISOTROPIC ELASTICITY AND ISOTROPIC PLASTICITY

C J2 FLOW THEORY

C
C PROPS(1) - E
C PROPS(2) - NU
C PROPS(3) - POWER - POWER LAW EXPONENT
C PROPS(4) - SIGO
C PROPS(5) - EPSO
C PROPS(6) - STTOL - "ELASTIC" STRAIN/"YIELD" STRAIN
C PROPS(7) - PNLT - PENALTY FOR INCOMPRESSIBILITY

C
C COMPUTE TOTAL STRAIN

C
DO K1=1,NTENS
STRANT(K1) = STRAN(K1)*DSTRAN(K1)
ENDDO

C
C COMPUTE MEAN STRAIN

C
STNMN = ZERO
DO K1 = 1,NDI
STNMN = STRANT(K1)*STRANT(K1)
ENDDO
DO K1 = 1+NDI,NTENS
STNMN = STRANT + HALF*STRANT(K1)*STRANT(K1)
ENDDO

ENDDO  

STNMN = (TWO/THREE*STNMN)**HALF

C

C ELASTIC PROPERTIES

C

EMOD=PROP5(1)

ENU=PROP5(2)

IF (ENU.GT.0.4999 AND.ENU.LT.0.5001) ENU=0.49

EBULK3=EMOD/(ONE-TWO*ENU)

EG2=EMOD/(ONE+ENU)

EG3=THREE*EG

ELAM=(EBULK3-EG2)/THREE

C

C MATERIAL PROPERTIES AND PENALTIES

C

POWER = PROPS(3)

SIGO = PROPS(4)

EPS0 = PROPS(5)

STTOL = PROPS(6)*EPS0

PNLT = PROPS(7)*EMOD

C

C SWITCH FOR LINEAR/POWER LAW BEHAVIOR

C IF (STNMN.LE.STTOL) THEN

C

C ELLASTIC STIFFNESS

C DO 20 K1=1,NTENS

DO 10 K2=1,NTENS

DDSDDE(K2,K1)=ZERO

10 CONTINUE

20 CONTINUE

C DO 40 K1=1,NDI

DO 30 K2=1,NDI

DDSDDE(K2,K1)=ELAM

30 CONTINUE

DDSDDE(K1,K1)=EG2+ELAM  

40 CONTINUE
DO 50 K1=NDI+1,NTENS
DDSDDE(K1,K1)=EG
CONTINUE
C
CALCULATE STRESS FROM ELASTIC STRAINS
C
DO 70 K1=1,NTENS
DO 60 K2=1,NTENS
STRESS(K2)=STRESS(K2)+DDSDDE(K2,K1)*DSTRAN(K1)
60 CONTINUE
C
CONTINUE
C
ELSE
C
C NON-LINEAR BEHAVIOR
C
C PRESSURE
C
VOLDEF = 0.0
DO K1 = 1,NDI
VOLDEF = VOLDEF + STRANT(K1)
ENDDO
PRESS = PNLT*VOLDEF
C
C PRECOMPUTED TERMS
C
TERM1 = TWO/THREE*(STNMN/EPSO)**(POWER-ONE)*SIG0/EPS0
TERM2 = (POWER-ONE)*TWO/THREE/(STNMN*STMNN)
TERM3 = HALF*TERM1*TERM2
C
C STRESS
C
DO K1 = 1,NDI
STRESS(K1) = TERM1*STRANT(K1) + PRESS
ENDDO
DO K1 = NDI+1, NTENS
STRESS(K1) = HALF*TERM1*STRANT(K1)
ENDDO  

C
C JACOBIAN
C
C
JACOBIAN - TENSION-TENSION COMPONENTS
C
1 DO K1 = 1,NDI
DO K2 = 1,NDI
DDSDDE(K1,K2) = TERM1*(TERM2*
STRANT(K1)*STRANT(K2)+DELTA(K1,K2)) + PNLT
ENDDO
ENDDO
C
C JACOBIAN - TENSION-SHEAR COMPONENTS
C
DO K1 = 1,NDI
DO K2 = NDI+1,NTENS
DDSDDE(K1,K2) = TERM3*STRANT(K1)*STRANT(K2)
DDSDDE(K2,K1) = DSSDDE(K1,K2)
ENDDO
ENDDO
C
C JACOBIAN - SHEAR-SHEAR COMPONENTS
C
DO K1 = NDI+1,NTENS
DO K2 = NDI+1,NTENS
DDSDDE(K1,K2) = HALF*TERM1* (HALF*TERM2*
STRANT(K1)*STRANT(K2)+DELTA(K1-NDI,K2-NDI))
ENDDO
ENDDO
ENDIF
C
RETURN
END
*BOUNDARY
1,PINNED
2,2
5,2
6,2  

4,1
5,1
8,1
2,3
3,3
4,3
*STEP, INC=100
*STATIC
1.,20.
*BOUNDARY
7,3,,1.
5,3,,1.
6,3,,1.
8,3,,1.
*EL PRINT
S
SINV
E
EE
*NODE PRINT
U,RF
*EL FILE,FREQ=10
S,E
*RESTART,WRITE
*END STEP  
```



## 4 ABAQUS BC: Shear Test  

```fortran
*BOUNDARY

1, 1, 3

4, 1, 3

5, 1, 3

8, 1, 3

2, 3, 3

3, 3, 3

6, 3, 3

7, 3, 3

2, 1, 1

3, 1, 1

6, 1, 1

7, 1, 1

*STEP, INC=100

*STATIC

1., 20.

*BOUNDARY

2, 2, 1.

3, 2, 1.

6, 2, 1.

7, 2, 1.

*EL PRINT  
```

5 Results for Uniaxial Tension Test   


| ELEMENT | PT FOOT-NOTE | MISES | TREC | PRESS | INV3 |
| --- | --- | --- | --- | --- | --- |
|  | 1 | 1.0000 | 1.0000 | -.3333 | 1.0000 |
|  | 1 | 2 | 1.0000 | 1.0000 | -.3333 | 1.0000 |
|  | 1 | 3 | 1.0000 | 1.0000 | -.3333 | 1.0000 |
|  | 1 | 4 | 1.0000 | 1.0000 | -.3333 | 1.0000 |
|  | 1 | 5 | 1.0000 | 1.0000 | -.3333 | 1.0000 |
|  | 1 | 6 | 1.0000 | 1.0000 | -.3334 | 1.0000 |
|  | 1 | 7 | 1.0000 | 1.0000 | -.3333 | 1.0000 |
|  | 1 | 8 | 1.0000 | 1.0000 | -.3333 | 1.0000 |
| MAXIMUM | 1.0000 | 1.0000 | -.3333 | 1.0000 |
| ELEMENT | 1 | 1 | 1 | 1 |  

## 6 Results for Shear Test  

| ELEMENT | PT FOOT-NOTE | MISES | TREC | PRESS | INV3 |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | .7598 | .8774 | -6.9389E-07 | 0.0000E+00 |
| 1 | 2 | .7598 | .8774 | -3.4694E-07 | 0.0000E+00 |
| 1 | 3 | .7598 | .8774 | -6.9389E-07 | 0.0000E+00 |
| 1 | 4 | .7598 | .8774 | -3.4694E-07 | 0.0000E+00 |
| 1 | 5 | .7598 | .8774 | -4.8572E-06 | 0.0000E+00 |
| 1 | 6 | .7598 | .8774 | 6.9389E-07 | 0.0000E+00 |
| 1 | 7 | .7598 | .8774 | 3.9031E-06 | 0.0000E+00 |
| 1 | 8 | .7598 | .8774 | 7.6328E-06 | 0.0000E+00 |
| MAXIMUM |  



![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/e5c445e2647e94997f858aa98a271792889e268e2f08e0eeac172f15d2f7db9d_65.jpg){width=65%}  
Figure 1: Uniaxial Tension Test  

![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/367bc1ed0015b22de10542596a34d117be184749a0061d9f60c35ffec8caa739_65.jpg){width=65%}  



## 7 Pressurized Pipe  

### 7.1 Model Definition for One Layer of Elements  

```fortran
*HEADING
INF. LONG CYLIN. TUBE SUBJECTED TO INT.PRESS - POWER LAW M
UNITS: N, mm
*RESTART,WRITE
*NODE
1, 60.0, 0.0
9, 0.0, 60.0
161, 140.0, 0.0
169, 0.0, 140.0
*NGEN,LINE=C,NSET=Di
1, 9, 1, , 0.0, 0.0, 0.0
*NGEN,LINE=C,NSET=Do
161, 169, 1, , 0.0, 0.0, 0.0
*NFILL,NSET=Face1
Di, Do, 16, 10
*NCOPY,CHANGE NUMBER=2000,OLD SET=Face1,SHIFT,NEW SET=Face2
0.0, 0.0, 300.0
0.0,
*NFILL,NSET=Nall
Face1, Face2, 2, 1000
*NSET,GENERATE,NSET=Xsymm
1, 161, 10
1001, 1161, 10
2001, 2161, 10
*NSET,GENERATE,NSET=Ysymm
9, 169, 10
1009, 1169, 10
2009, 2169, 10
*NSET,GENERATE,NSET=Noutp
1, 161, 10

***

*ELEMENT,TYPE=C3D20R
1, 1, 161, 163, 3, 2001, 2161, 2163, 2003, 81, 162, 8
2083, 2002, 1001, 1161, 1163, 1003
*ELGEN,ELSET=Eall  

1, 4, 2, 1

*ELSET,GENERATE,ELSET=Inside

1, 4

*ELSET,ELSET=Eoutp

1

*SOLID SECTION,ELSET=Eall,MATERIAL=ALLE

*MATERIAL,NAME=ALLE

*USER MATERIAL,CONSTANTS=7

**E v POWER sig0 eps0 StTol Pnlt

200.E3,.3,.5,1..,1..,1.E-6,1.E6

*USER SUBROUTINE

SUBROUTINE UMAT(STRESS,STATEV,DDSDDE,SSE,SPD,SCD,

. . .

END

*BOUNDARY

Xsymm, 2

Ysymm, 1

Face1, 3

Face2, 3

*

*STEP,PERTURBATION

*STATIC

*DLOAD

Inside, P6, 50.0

*EL PRINT,ELSET=Eoutp

1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16

COORD, S11, S22, S33

*EL PRINT,ELSET=Eoutp

17,18,19,20,21,22,23,24

COORD, S11, S22, S33

*EL FILE

S, E

*NODE PRINT,NSET=Noutp

U1,

*NODE FILE

U, RF

*END STEP

*END STEP 
```

 

### 7.2 Model Definition for Four Layers of Elements  

```fortran
*HEADING
INF. LONG CYLIN. TUBE SUBJECTED TO INT.PRESS - POWER LAW M
UNITS: N, mm
*RESTART,WRITE
*
*NODE
1, 60.0, 0.0
9, 0.0, 60.0
161, 140.0, 0.0
169, 0.0, 140.0
*NGEN,LINE=C,NSET=Di
1, 9, 1, , 0.0, 0.0, 0.0
*NGEN,LINE=C,NSET=Do
161, 169, 1, , 0.0, 0.0, 0.0
*NFILL,NSET=Face1
Di, Do, 16, 10
*NCOPY,CHANGE NUMBER=2000,OLD SET=Face1,SHIFT,NEW SET=Face2
0.0, 0.0, 300.0
0.0,
*NFILL,NSET=Nall
Face1, Face2, 2, 1000
*NSET,GENERATE,NSET=Xsymm
1, 161, 10
1001, 1161, 10
2001, 2161, 10
*NSET,GENERATE,NSET=Ysymm
9, 169, 10
1009, 1169, 10
2009, 2169, 10
*NSET,GENERATE,NSET=Noutp
1, 161, 10

***

*ELEMENT,TYPE=C3D20R
1, 1, 41, 43, 3, 2001, 2041, 2043, 2003, 21, 42, 23,
2023, 2002, 1001, 1041, 1043, 1003
*ELGEN,ELSET=Eall
1, 4, 2, 1, 4, 40, 10
*ELSET,GENERATE,ELSET=Inside  

1, 4

*ELSET,GENERATE,ELSET=Eoutp

1, 41, 10

*SOLID SECTION,ELSET=Eall,MATERIAL=ALLE

*MATERIAL,NAME=ALLE

*USER MATERIAL,CONSTANTS=7

**E v POWER sig0 eps0 StTo1 Pnlt

200.E3,.3,.5,1..,1..,1.E-6,1.E6

*USER SUBROUTINE

SUBROUTINE UMAT(STRESS,STATEV,DDSDDE,SSE,SPD,SCD,

. . .

END

*BOUNDARY

Xsymm, 2

Ysymm, 1

Face1, 3

Face2, 3

*

*STEP,PERTURBATION

*STATIC

*DLOAD

Inside, P6, 50.0

*EL PRINT,ELSET=Eoutp

1,2,3,4,5,6

COORD, S11, S22, S33

*EL FILE

S, E

*NODE PRINT,NSET=Noutp

U1

*NODE FILE

U,RF

*END STEP
```

  



![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/478728f2fe6f3dac3a2fb029b4de472a34e0f17433b0903ca9de4e64c1c15c1d_65.jpg){width=65%}  
Figure 3: Pressurized Pipe. One layer of elements. Power Law Material. Mesh.  

![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/ff47792170b1c4d4d1735d1a915723787d714fa56e394fbac4cfe5085ee04140_65.jpg){width=65%}  
Figure 4: Pressurized Pipe. One layer of elements. Power Law Material.Displacement.  

![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/b77156cd6792df854048f15553c2bf217429e95d8c0daddd769630a7e26f39a5_64.jpg){width=64%}  
Figure 5: Pressurized Pipe. One layer of elements. Power Law Material. $\sigma_{22}$.  

![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/89f512d6018ca97cde6d1e06e5143c234aff665de2b78d1e275befccb8e733e9_65.jpg){width=65%}  
Figure 6: Pressurized Pipe. Four layers of elements. Power Law Material. Mesh.  



![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/a024ec953d946c5dabeb371693cc8977cd127f33f1a1913cdcc2bba10a6d9dfe_65.jpg){width=65%}  
Figure 7: Pressurized Pipe. Four layers of elements. Power Law Material.Displacement.  

![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/bfddd0f17619de22199ade526dfffc805bdbe2b1b6f5159eec5b1efd255a1863_65.jpg){width=65%}  
Figure 8: Pressurized Pipe. Four layers of elements. Power Law Material. $\sigma_{22}$.  



![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/ff113919403bf77ceea29472166b98edf14a85f272a2d81912b32b3a4e1c04e6_64.jpg){width=64%}  
Figure 9: Pressurized Pipe. Four layers of elements. Linear Material. Displacement.  

![](https://cdn.jsdelivr.net/gh/LiZhenxing1998/bolg-img@main/cf40af4f7b87d76bc53c915325c9a26e8c960191257c60721cdadb83db3b4e9d_65.jpg){width=65%}  
Figure 10: Pressurized Pipe. Four layers of elements. Linear Material.$\sigma_{22}$.  