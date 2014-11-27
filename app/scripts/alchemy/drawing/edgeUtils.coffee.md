    # Alchemy.js is a graph drawing application for the web.
    # Copyright (C) 2014  GraphAlchemist, Inc.

    # This program is free software: you can redistribute it and/or modify
    # it under the terms of the GNU Affero General Public License as published by
    # the Free Software Foundation, either version 3 of the License, or
    # (at your option) any later version.

    # This program is distributed in the hope that it will be useful,
    # but WITHOUT ANY WARRANTY; without even the implied warranty of
    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    # GNU Affero General Public License for more details.

    # You should have received a copy of the GNU Affero General Public License
    # along with this program.  If not, see <http://www.gnu.org/licenses/>.

    Alchemy::EdgeUtils = (instance)->
        a: instance
        edgeStyle: (d) ->
            conf = @a.conf
            edge = @a._edges[d.id][d.pos]
            styles = @a.svgStyles.edge.update edge
            nodes = @a._nodes

            # edge styles based on clustering
            if @a.conf.cluster
                clustering = @a.layout._clustering
                styles.stroke = do (d) ->
                    clusterKey = conf.clusterKey
                    source = nodes[d.source.id]._properties
                    target = nodes[d.target.id]._properties
                    if source.root or target.root
                        index = if source.root then target[clusterKey] else source[clusterKey]
                        "#{clustering.getClusterColour(index)}"
                    else if source[clusterKey] is target[clusterKey]
                        index = source[clusterKey]
                        "#{clustering.getClusterColour(index)}"
                    else if source[clusterKey] isnt target[clusterKey]
                        # use gradient between the two clusters' colours
                        id = "#{source[clusterKey]}-#{target[clusterKey]}"
                        gid = "cluster-gradient-#{id}"
                        "url(##{gid})"

            styles

        triangle: (edge) ->
            width = edge.target.x - edge.source.x
            height = edge.target.y - edge.source.y
            hyp = Math.sqrt height * height + width * width
            [width, height, hyp]

This is the primary function used to draw the svg paths between
two nodes for directed or undirected noncurved edges. 

        edgeWalk: (edge) ->
            [width, height, hyp] = @triangle edge

            edgeWidth = edge['stroke-width']

            curveOffset = 2

            startPathX = edge.source.radius + edge.source['stroke-width'] - (edgeWidth / 2) + curveOffset
            edgeLength = hyp - startPathX - curveOffset * 1.5

            edgeAngle: Math.atan2(height, width) / Math.PI * 180
            edgeLength: edgeLength

        middleLine: (edge) -> @curvedDirectedEdgeWalk edge, 'middle'
        startLine: (edge) -> @curvedDirectedEdgeWalk edge, 'linkStart'
        endLine: (edge) -> @curvedDirectedEdgeWalk edge, 'linkEnd'
        
        edgeLength: (edge) ->
            # build a right triangle
            width  = edge.target.x - edge.source.x
            height = edge.target.y - edge.source.y
            # as in hypotenuse 
            hyp = Math.sqrt(height * height + width * width)
        edgeAngle: (edge) ->
            width  = edge.target.x - edge.source.x
            height = edge.target.y - edge.source.y
            Math.atan2(height, width) / Math.PI * 180
        
        captionAngle: (angle) ->
            if angle < -90 or angle > 90
                180
            else
                0
        middlePath: (edge) ->
            pathNode = @a.vis
                         .select "#path-#{edge.id}"
                         .node()
            midPoint = pathNode.getPointAtLength pathNode.getTotalLength()/2
 
            x: midPoint.x
            y: midPoint.y
                
        # Temporary fill in for curved edges until math is completed for new path only edges
        middlePathCurve: (edge) ->
            pathNode = d3.select("#path-#{edge.id}").node()
            midPoint = pathNode.getPointAtLength(pathNode.getTotalLength()/2)

            x: midPoint.x
            y: midPoint.y