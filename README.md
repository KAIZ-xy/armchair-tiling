This project is about tilings
     
     
     
     import java.awt.*;
     import java.awt.geom.Path2D;
     import java.util.ArrayList;
     import java.util.List;
     import javax.swing.*;
     import static java.lang.Math.*;
     import static java.util.stream.Collectors.toList;

        public class PenroseTiling extends JPanel {
        class Tile {
        double x, y, angle, size;
        Type type;

        Tile(Type t, double x, double y, double a, double s) {
            type = t;
            this.x = x;
            this.y = y;
            angle = a;
            size = s;
        }

        @Override
        public boolean equals(Object o) {
            if (o instanceof Tile) {
                Tile t = (Tile) o;
                return type == t.type && x == t.x && y == t.y && angle == t.angle;
            }
            return false;
        }
    }

    enum Type {
        L1, L2
    }
    
    static final double G = (2); // generate factor
    static final double T = toRadians(90); // theta

    List<Tile> tiles = new ArrayList<>();

    public PenroseTiling() {
        int w = 700, h = 450;
        setPreferredSize(new Dimension(w, h));
        setBackground(Color.white);
        tiles = deflateTiles(setupPrototiles(w, h), 2); // Increased number of generations
    }

    List<Tile> setupPrototiles(int w, int h) {
        List<Tile> proto = new ArrayList<>();
        proto.add(new Tile(Type.L2, w / 2, h / 2, 0, w / 10)); // Increased initial size
        return proto;
    }

    List<Tile> deflateTiles(List<Tile> tls, int generation) {
        if (generation <= 0) return tls;
        List<Tile> next = new ArrayList<>();
        for (Tile tile : tls) {
        	double x = tile.x, y = tile.y, a = tile.angle, nx, ny,mx,my,ox,oy;
        	double size = tile.size / G;
        	if (tile.type == Type.L1) {
        	next.add(new Tile(Type.L2, x, y, a, size)); {
        	for (int i = 0, sign = 1; i < 2; i++, sign *= -1) {
        	nx = x + cos(a - 3*T * sign) * tile.size * G;
            ny = y - sin(a - 2*T* sign) * tile.size * G;
    	    next.add(new Tile(Type.L1, nx, ny, a + T* sign, size));
    	    mx = x + cos(a - 3*T* sign) * tile.size * G;
    	    my = y - sin(a - 2*T* sign) * tile.size * G;
    	    next.add(new Tile(Type.L1, mx, my, a + 2 * T * sign, size));
    	    ox = x + cos(a + 3*T* sign) * tile.size * G;
    	    oy = y - sin(a - 4*T* sign) * tile.size * G;
    	    next.add(new Tile(Type.L2, ox, oy, a + 3 * T * sign, size));}
        	}} else {
        	for (int i = 0, sign = 1; i < 2; i++, sign *= -1) {
        	next.add(new Tile(Type.L1, x, y, a * sign, size));
        	nx = x + cos(a - 3*T * sign) * tile.size * G;
            ny = y - sin(a - 2*T* sign) * tile.size * G;
        	next.add(new Tile(Type.L2, nx, ny, a + T* sign, size));
        	mx = x + cos(a - 3*T* sign) * tile.size * G;
    	    my = y - sin(a - 2*T* sign) * tile.size * G;
    	    next.add(new Tile(Type.L2, mx, my, a + 2 * T * sign, size));
    	    ox = x + cos(a + 3*T* sign) * tile.size * G;
    	    oy = y - sin(a - 4*T* sign) * tile.size * G;
    	    next.add(new Tile(Type.L1, ox, oy, a + 3 * T * sign, size));
        	}
        	}
        	}
        tls = next.stream().distinct().collect(toList());
        return deflateTiles(tls, generation - 1);
    }

    void drawTiles(Graphics2D g) {
        for (Tile tile : tiles) {
            double angle = tile.angle - PI/2;
            double side1 = tile.size;
            double side2 = (2.0 / 3.0) * tile.size;
            double side3 = (1.0 / 3.0) * tile.size;

            Path2D path = new Path2D.Double();

            // Vertices of the L shape
            double[][] vertices = new double[6][2];
            vertices[0][0] = tile.x; // x
            vertices[0][1] = tile.y; // y
            vertices[1][0] = tile.x + cos(angle + T) * side2;
            vertices[1][1] = tile.y - sin(angle + T) * side2;
            vertices[2][0] = vertices[1][0] + cos(angle + T*2) * side3;
            vertices[2][1] = vertices[1][1] - sin(angle + T*2) * side3;
            vertices[3][0] = vertices[2][0] + cos(angle + T*3) * side3;
            vertices[3][1] = vertices[2][1] - sin(angle + T*3) * side3;
            vertices[4][0] = vertices[3][0] + cos(angle + T*2) * side2;
            vertices[4][1] = vertices[3][1] - sin(angle + T*2) * side2;
            vertices[5][0] = vertices[4][0] + cos(angle + T*3) * side3;
            vertices[5][1] = vertices[4][1] - sin(angle + T*3) * side3;

            // Move to the first vertex
            path.moveTo(vertices[0][0], vertices[0][1]);

            // Draw lines to each subsequent vertex
            for (int i = 1; i < vertices.length; i++) {
                path.lineTo(vertices[i][0], vertices[i][1]);
            }

            path.closePath();
            g.setColor(tile.type == Type.L1 ? Color.yellow : Color.orange);
            g.fill(path);
            g.setColor(Color.darkGray);
            g.draw(path);
        }
    }

    @Override
    public void paintComponent(Graphics og) {
        super.paintComponent(og);
        Graphics2D g = (Graphics2D) og;
        g.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        g.translate(200, 280);
        g.scale(0.75, 0.75); // Adjusted scaling factor to fit shapes in the window
        drawTiles(g);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            JFrame f = new JFrame();
            f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            f.setTitle("Penrose Tiling");
            f.setResizable(true);
            f.add(new PenroseTiling(), BorderLayout.CENTER);
            f.pack();
            f.setLocationRelativeTo(null);
            f.setVisible(true);
        });
    }
}

