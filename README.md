# THICKNESS-SCRIPT
#THICKNESS SCRIPT FOR HYPERMESH

#################################################################
# File               : thicknessTags.tcl
# Date               : 
# Created by: 
# Purpose: 
#################################################################
#thicknessTags.tcl 
   variable ::numNodes;
   set ::numNodes 3;
   catch {destroy .thicknessTagsDot}
    set top .thicknessTagsDot
    ###################
    # CREATING WIDGETS
    ###################
    toplevel $top -class Toplevel
    wm focusmodel $top passive
    wm geometry $top 190x220+719+543; update
    wm maxsize $top 190 220
    wm minsize $top 115 1
    wm overrideredirect $top 0
    wm resizable $top 1 1
    wm deiconify $top
    wm title $top "Thickness Tags Average"

    button $top.but46 \
        -pady 0 -text "Line to Surface" -command {thicknessTagCreate 2}
    button $top.but47 \
        -pady 0 -text "Line to Line" -command {thicknessTagCreate 1}
    button $top.but49 \
        -pady 0 -text "Nodes to Surface" -command {thicknessTagCreate 3}
		
	button $top.but48 \
        -pady 0 -text "DeleteTags" -command {deleteTagsAll}
	entry $top.ent01 \
		-textvariable "::numNodes";
    ###################
    # SETTING GEOMETRY
    ###################
    place $top.but46 \
        -in $top -x 30 -y 15 -width 90 -height 30 -anchor nw \
        -bordermode ignore 
    place $top.but47 \
        -in $top -x 30 -y 55 -width 90 -height 30 -anchor nw \
        -bordermode ignore 
    place $top.but49 \
        -in $top -x 30 -y 95 -width 90 -height 30 -anchor nw \
        -bordermode ignore 
    place $top.but48 \
        -in $top -x 30 -y 135 -width 90 -height 30 -anchor nw \
        -bordermode ignore 
    place $top.ent01 \
        -in $top -x 30 -y 175 -width 90 -height 30 -anchor nw \
        -bordermode ignore 

		
proc Bool_NodeShift {nodeid1 nodeid2} {
set xn1 [hm_getentityvalue NODES $nodeid1 "x" 0]
set yn1 [hm_getentityvalue NODES $nodeid1 "y" 0]
set zn1 [hm_getentityvalue NODES $nodeid1 "z" 0]

set xn2 [hm_getentityvalue NODES $nodeid2 "x" 0]
set yn2 [hm_getentityvalue NODES $nodeid2 "y" 0]
set zn2 [hm_getentityvalue NODES $nodeid2 "z" 0]

if {$xn1 == $xn2 && $yn1 == $yn2 && $zn1 == $zn2} {
return 1
} else {
return 0
}
}

proc deleteTagsAll { } {
*createmark tags 1 all
catch {*deletemark tags 1}
*clearmark nodes 1;
*clearmark nodes 2;
*clearmark surfs 1;
*clearmark surfs 2;
*nodecleartempmark
}


proc getDistanceFromTwoPoints {nodeid1 nodeid2} {
set xn1 [hm_getentityvalue NODES $nodeid1 "x" 0]
set yn1 [hm_getentityvalue NODES $nodeid1 "y" 0]
set zn1 [hm_getentityvalue NODES $nodeid1 "z" 0]

set xn2 [hm_getentityvalue NODES $nodeid2 "x" 0]
set yn2 [hm_getentityvalue NODES $nodeid2 "y" 0]
set zn2 [hm_getentityvalue NODES $nodeid2 "z" 0]

set dx1 [ expr $xn1 - $xn2];
set dy1 [ expr $yn1 - $yn2];
set dz1 [ expr $zn1 - $zn2];
return [ expr sqrt(($dx1 * $dx1 ) + ( $dy1 * $dy1 ) + ( $dz1 * $dz1 )) ];
}



proc PromptAndGetSelectedEntity {entity text} {

variable selectcheck
::hwt::UnpostWindow .thicknessTagsDot
    set selectcheck "true"
    *clearmark $entity 1
    
    while {$selectcheck} {
        *createmarkpanel $entity 1 $text
        set ids [hm_getmark $entity 1]
       
        if {$ids == {}} {
            set response [tk_messageBox -icon error -message "Press retry to select the $entity or cancel to abort" \
                                        -type retrycancel]
            if {$response == "cancel"} {
                set selectcheck "false"
                set ids 0
            }
        } else {
            set selectcheck "false"
        }
    }
	::hwt::PostWindow .thicknessTagsDot
    return $ids
}

proc getmidCoordinatesfromTwoPoints {nodeid1 nodeid2} {
set xn1 [hm_getentityvalue NODES $nodeid1 "x" 0]
set yn1 [hm_getentityvalue NODES $nodeid1 "y" 0]
set zn1 [hm_getentityvalue NODES $nodeid1 "z" 0]

set xn2 [hm_getentityvalue NODES $nodeid2 "x" 0]
set yn2 [hm_getentityvalue NODES $nodeid2 "y" 0]
set zn2 [hm_getentityvalue NODES $nodeid2 "z" 0]

set x3 [ expr $xn1 + (($xn2-$xn1)/2)];
set y3 [ expr $yn1 + (($yn2-$yn1)/2)];
set z3 [ expr $zn1 + (($zn2-$zn1)/2)];

return [list $x3 $y3 $z3];
}


proc setnodenumbers {nos} {
set var ""
for {set i 1} {$i<=[expr $nos+0]} {incr i} {
   append var " -" $i
}
return $var
}

proc thicknessTagCreate {selectFlag} {
   variable ::numNodes;
set nosNodes $::numNodes;

*clearmark nodes 1;
*clearmark nodes 2;
*clearmark surfs 1;
*clearmark surfs 2;

if {$selectFlag == 1} {
*nodecleartempmark
set line1ID [PromptAndGetSelectedEntity lines "Select 1 Line to create thickness tags"]
set line2ID [PromptAndGetSelectedEntity lines "Select 2 Line to create thickness tags"]

if { ! [ Null line2ID ] } {
if { ! [ Null line1ID ] } {
		if {[llength $line1ID] == 1 && [llength $line2ID] == 1} {
			if {$line1ID != $line2ID} {
			*createmark lines 1 $line1ID;
			*nodecreateonlines lines 1 $nosNodes 0 0
			set nodeTomarkMinusSign [setnodenumbers $nosNodes]
			#tk_messageBox -message "nodeTomarkMinusSign=$nodeTomarkMinusSign";
			eval *createmark nodes 1 $nodeTomarkMinusSign;
			set NodesToproject [hm_getmark nodes 1];
			#tk_messageBox -message "NodesToproject=$NodesToproject";
			*clearmark nodes 1;
			set totalDist 0;
			set inc 0;
			foreach pronode $NodesToproject {
			*createmark nodes 1 $pronode;
			*duplicatemark nodes 1 0;
			*createmark nodes 1 -1;
			set projectedNodeDupli [hm_getmark nodes 1]
			*clearmark nodes 1;
			*createmark lines 1 $line2ID;
			*nodecreateonlines lines 1 2
			#tk_messageBox -message "projectedNodeDupli=$projectedNodeDupli";
					*createmark nodes 1 $projectedNodeDupli
					*createmark nodes 2 -1 -2
					*createlist lines 2 $line2ID
					set twonod [hm_getmark nodes 2]
					#tk_messageBox -message "$twonod"
					
					*createlist nodes 2 $twonod
					*markprojectnormallytoline nodes 1 2 2
					*clearmark nodes 1;
					*nodemarkcleartempmark 2
					#*createmark nodes 1 -1;
					set projectedNode $projectedNodeDupli;
					#tk_messageBox -message "Bool_NodeShift=[Bool_NodeShift $pronode $projectedNode]";
					set checkbool [Bool_NodeShift $pronode $projectedNode]
					
					if { $checkbool == 0} {
					set midCords [getmidCoordinatesfromTwoPoints $pronode $projectedNode]
					set distance [format "%.2f" [getDistanceFromTwoPoints $pronode $projectedNode]]
					set totalDist [expr $totalDist+$distance];
					incr inc;
					*createnode [lindex $midCords 0] [lindex $midCords 1] [lindex $midCords 2] 0 0 0;
					*createmark node 1 -1;
					set midnode [hm_getmark node 1];
					*tagcreate nodes $midnode "$distance" "$midnode" 2
					
					} else {
					continue;
					}
			}
			  set midNumber [expr $::numNodes / 2]
				set midNumberNode [lindex $NodesToproject $midNumber]
				set average [format "%.2f" [expr $totalDist/$inc]]
				*tagcreate nodes $midNumberNode "Average=$average" "$midNumberNode" 6
			
			} else {
			tk_messageBox -message "You cannot select both entities as same.";
			return;
			}
		} else {
		tk_messageBox -message "you just select only one entity.";
		return;
		}
		#$*nodecleartempmark
} else {
return;
}
} else {
return;
}
}

if {$selectFlag == 2} {
set line1ID [PromptAndGetSelectedEntity lines "Select 1 Line to create thickness tags"]
set surfID [PromptAndGetSelectedEntity surfaces "Select 1 surface to create thickness tags"]

if { ! [ Null line1ID ] } {
		if {[llength $line1ID] == 1 && [llength $surfID] == 1} {
			if {$line1ID != $surfID} {
			*createmark lines 1 $line1ID;
			*nodecreateonlines lines 1 $nosNodes 0 0
			set nodeTomarkMinusSign [setnodenumbers $nosNodes]
			#tk_messageBox -message "nodeTomarkMinusSign=$nodeTomarkMinusSign";
			eval *createmark nodes 1 $nodeTomarkMinusSign;
			set NodesToproject [hm_getmark nodes 1];
			#tk_messageBox -message "NodesToproject=$NodesToproject";
			*clearmark nodes 1;
			set totalDist 0;
			set inc 0;
			foreach pronode $NodesToproject {
			*createmark nodes 1 $pronode;
			*duplicatemark nodes 1 0;
			*createmark nodes 1 -1;
			set projectedNodeDupli [hm_getmark nodes 1]
			#*clearmark nodes 1;

			#tk_messageBox -message "projectedNodeDupli=$projectedNodeDupli";
					#*createmark nodes 1 $projectedNodeDupli
					*markprojectnormallytosurface nodes 1 $surfID;
					*clearmark nodes 1;
					#*createmark nodes 1 -1;
					set projectedNode $projectedNodeDupli;
					#tk_messageBox -message "Bool_NodeShift=[Bool_NodeShift $pronode $projectedNode]";
					set checkbool [Bool_NodeShift $pronode $projectedNode]
					
					if { $checkbool == 0} {
					set midCords [getmidCoordinatesfromTwoPoints $pronode $projectedNode]
					#tk_messageBox -message "checkbool=$checkbool";
					set distance [format "%.2f" [getDistanceFromTwoPoints $pronode $projectedNode]]
					set totalDist [expr $totalDist+$distance];
					incr inc;
					*createnode [lindex $midCords 0] [lindex $midCords 1] [lindex $midCords 2] 0 0 0;
					*createmark node 1 -1;
					set midnode [hm_getmark node 1];
					*tagcreate nodes $midnode "$distance" "$midnode" 2
					} else {
					continue;
					}
			}
			  set midNumber [expr $::numNodes / 2]
				set midNumberNode [lindex $NodesToproject $midNumber]
				#tk_messageBox -message "$totalDist $inc";
				if {$totalDist == 0 || $inc == 0} {
				return;
				}
				set average [format "%.2f" [expr $totalDist/$inc]]
				*tagcreate nodes $midNumberNode "Average=$average" "$midNumberNode" 6
			
			} else {
			tk_messageBox -message "You cannot select both entities as same.";
			return;
			}
		} else {
		tk_messageBox -message "you just select only one entity.";
		return;
		}
		#$*nodecleartempmark
} else {
return;
}
}



if {$selectFlag == 3} {
set nodeID [PromptAndGetSelectedEntity nodes "Select nodes to create thickness tags"]
set surfID [PromptAndGetSelectedEntity surfaces "Select 1 surface to create thickness tags"]

if { ! [ Null nodeID ] } {
		if {[llength $surfID] == 1} {
			set NodesToproject $nodeID
			foreach pronode $NodesToproject {
	  			*createmark nodes 1 $pronode;
	  			*duplicatemark nodes 1 0;	  			
     			*duplicatemark nodes 1 0;
	        *createmark nodes 1 -1;
     			set projectedNodeDupli [hm_getmark nodes 1]
     			*markprojectnormallytosurface nodes 1 $surfID;
			    *clearmark nodes 1;
					set projectedNode $projectedNodeDupli;
					#tk_messageBox -message "Bool_NodeShift=[Bool_NodeShift $pronode $projectedNode]";
					set checkbool [Bool_NodeShift $pronode $projectedNode]
					
					if { $checkbool == 0} {
					set midCords [getmidCoordinatesfromTwoPoints $pronode $projectedNode]
					set distance [format "%.2f" [getDistanceFromTwoPoints $pronode $projectedNode]]
					*createnode [lindex $midCords 0] [lindex $midCords 1] [lindex $midCords 2] 0 0 0;
					*createmark node 1 -1;
					set midnode [hm_getmark node 1];
					*tagcreate nodes $midnode "Distance=$distance" "$midnode" 6
					
					} else {
					continue;
					}
     
         }          					

					} else {
			     tk_messageBox -message "You cannot select both entities as same.";
			     return;
			   }
			} else {
      return;
      }          					
  *createmark surfs 1 all
  hm_highlightmark surfs 1 "norm"
  }



}


