# topology
module Topo {
    class Util{
        static forEach(obj, iteratee,context?){
            if (obj == null) return obj;
            var i, length = obj.length;
            if (length === +length) {
                for (i = 0; i < length; i++) {
                    if (iteratee.call(context, i, obj[i]) == true) {
                        return;
                    }
                }
            } else {
                var ks = Object.keys(obj);
                var keys = [];
                for (i = 0, length = ks.length; i < length; i++) {
                    if (obj.hasOwnProperty(ks[i])) {
                        keys.push(ks[i]);
                    }
                }
                for (i = 0, length = keys.length; i < length; i++) {
                    if (iteratee.call(context, keys[i], obj[keys[i]]) == true) {
                        return;
                    }
                }
            }
            return obj;
        }
    }
    class Point{
        private _x:number;
        private _y:number;
        constructor(x:number,y:number){
            this.x = x;
            this.y = y;
        }
        public get x():number {
            return this._x;
        }

        public set x(value:number) {
            this._x = value;
        }

        public get y():number {
            return this._y;
        }

        public set y(value:number) {
            this._y = value;
        }
    }
    class Element {
        private _network:Network;
        private _id:number;
        private _name:string;
        private _type:ElementType;
        private _ctx:CanvasRenderingContext2D;

        paint() {

        }

        public get ctx():CanvasRenderingContext2D {
            return this._ctx;
        }

        public set ctx(value:CanvasRenderingContext2D) {
            this._ctx = value;
        }

        public get network():Network {
            return this._network;
        }

        public set network(value:Network) {
            this._network = value;
        }

        public get type():ElementType {
            return this._type;
        }

        public set type(value:ElementType) {
            this._type = value;
        }

        public get id():number {
            return this._id;
        }

        public set id(value:number) {
            this._id = value;
        }

        public get name():string {
            return this._name;
        }

        public set name(value:string) {
            this._name = value;
        }


    }
    enum ElementType{
        NODE,
        LINK
    }
    var IdGenerator = {
        id: 0,
        next: function () {
            return this.id++;
        }
    };
    export class Node extends Element {
        private _x:number;
        private _y:number;
        private _width:number;
        private _height:number;

        constructor(name:string) {
            super();
            this.name = name;
            this.type = ElementType.NODE;
            this.id = IdGenerator.next();
        }
        containsPoint(point:Point) {
            if (point.x >= this._x && point.x <= this.x + this.width
            && point.y >= this._y && point.y <= this.y + this.height) {
                return true;
            }else{
                return false;
            }
        }
        paint() {
            this.network.context.fillStyle = "green";
            this.ctx.fillRect(this.x - this.width / 2,
                this.y - this.height / 2,
                this.width,
                this.height);
        }

        public get x():number {
            return this._x;
        }

        public set x(value:number) {
            this._x = value;
        }

        public get y():number {
            return this._y;
        }

        public set y(value:number) {
            this._y = value;
        }

        public get width():number {
            return this._width;
        }

        public set width(value:number) {
            this._width = value;
        }

        public get height():number {
            return this._height;
        }

        public set height(value:number) {
            this._height = value;
        }
    }
    export class Link extends Element {
        private _src:Node;
        private _dst:Node;

        constructor(name:string, src:Node, dst:Node) {
            super();
            this.id = IdGenerator.next();
            this.type = ElementType.LINK;
            this.src = src;
            this.dst = dst;
            this.name = name;
        }

        paint() {
            this.ctx.beginPath();
            this.ctx.lineWidth = 1;
            this.ctx.strokeStyle = "red"; // ºìÉ«Â·¾¶
            this.ctx.moveTo(this.src.x, this.src.y);
            this.ctx.lineTo(this.dst.x, this.dst.y);
            this.ctx.stroke(); // ½øÐÐ»æÖÆ
        }


        public get src():Node {
            return this._src;
        }

        public set src(value:Node) {
            this._src = value;
        }

        public get dst():Node {
            return this._dst;
        }

        public set dst(value:Node) {
            this._dst = value;
        }
    }

    export class Network {
        dataBox:DataBox;
        context:CanvasRenderingContext2D;
        canvas: HTMLCanvasElement;
        selectedNodes:Node[];
        lastPoint:Point;
        constructor(canvasId:string, dataBox:DataBox) {
            this.dataBox = dataBox;
            this.dataBox.network = this;
            this.canvas = <HTMLCanvasElement>document.getElementById(canvasId);
            this.context = this.canvas.getContext("2d");
            this.display();
        }
        repaint(){
            var id2Node = this.dataBox.id2Node;
            var id2Link = this.dataBox.id2Link;
            var nodeIds = Object.keys(id2Node);
            var linkIds = Object.keys(id2Link);

            this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
            var nodeId, linkId, node, link;
            for (var i = 0, length = linkIds.length; i < length; i++) {
                linkId = linkIds[i];
                link = id2Link[linkId];
                link.paint();
            }
            for (var i = 0, length = nodeIds.length; i < length; i++) {
                nodeId = nodeIds[i];
                node = id2Node[nodeId];
                node.paint();
            }
        }
        private display() {
            window.requestAnimationFrame(()=>{
                this.repaint.call(this);
            });
            this.canvas.addEventListener("mousedown", (event)=> {
                this.selectedNodes = [];
                var point = new Point(event.clientX, event.clientY);
                Util.forEach(this.dataBox.id2Node, (key:number, node:Node)=> {
                    if (node.containsPoint(point)){
                        this.selectedNodes.push(node);
                    }
                },this);
                this.lastPoint = point;
            });
            this.canvas.addEventListener("mousemove", (event)=> {
                var newPoint = new Point(event.clientX, event.clientY);
                var offsetX = newPoint.x - this.lastPoint.x;
                var offsetY = newPoint.y - this.lastPoint.y;
                this.lastPoint = newPoint;
                Util.forEach(this.selectedNodes, (key:number, node:Node)=> {
                    node.x += offsetX;
                    node.y += offsetY;
                });
                requestAnimationFrame(()=>{
                    this.repaint.call(this);
                });
            });
            this.canvas.addEventListener("mouseup", ()=> {
                this.lastPoint = null;
            });
        }
    }
    export class DataBox {
        private _id2Node:Object = {};
        private _id2Link:Object = {};
        private _network:Network;

        constructor() {
        }

        add(element:Element) {
            if (element.type == ElementType.NODE) {
                this.id2Node[element.id] = element;
                element.network = this.network;
                element.ctx = this.network.context;
            } else if (element.type == ElementType.LINK) {
                this.id2Link[element.id] = element;
                element.network = this.network;
                element.ctx = this.network.context;
            }
        }


        public get network():Network {
            return this._network;
        }

        public set network(value:Network) {
            this._network = value;
        }


        public get id2Node():Object {
            return this._id2Node;
        }

        public set id2Node(value:Object) {
            this._id2Node = value;
        }

        public get id2Link():Object {
            return this._id2Link;
        }

        public set id2Link(value:Object) {
            this._id2Link = value;
        }
    }
}
