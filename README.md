#include <GLFW/glfw3.h>
#include <cmath>
#include <vector>
#include <cstdlib>

struct NPC {
    float x,z;
};

struct Bullet {
    float x,z,dir;
};

float playerX=0;
float playerZ=0;
float angle=0;
bool inCar=false;

float carX=5;
float carZ=5;

std::vector<NPC> npcs;
std::vector<Bullet> bullets;

void cube(float x,float y,float z,float s){
    glPushMatrix();
    glTranslatef(x,y,z);
    glScalef(s,s,s);

    glBegin(GL_QUADS);

    // top
    glVertex3f(-1,1,-1);
    glVertex3f(1,1,-1);
    glVertex3f(1,1,1);
    glVertex3f(-1,1,1);

    // bottom
    glVertex3f(-1,-1,-1);
    glVertex3f(1,-1,-1);
    glVertex3f(1,-1,1);
    glVertex3f(-1,-1,1);

    // sides
    glVertex3f(-1,-1,-1);
    glVertex3f(-1,1,-1);
    glVertex3f(-1,1,1);
    glVertex3f(-1,-1,1);

    glVertex3f(1,-1,-1);
    glVertex3f(1,1,-1);
    glVertex3f(1,1,1);
    glVertex3f(1,-1,1);

    glEnd();

    glPopMatrix();
}

int main(){

    glfwInit();
    GLFWwindow* window=glfwCreateWindow(1000,700,"Mini GTA Sandbox",NULL,NULL);
    glfwMakeContextCurrent(window);

    glEnable(GL_DEPTH_TEST);

    // spawn npcs
    for(int i=0;i<20;i++){
        NPC n;
        n.x=(rand()%100)-50;
        n.z=(rand()%100)-50;
        npcs.push_back(n);
    }

    while(!glfwWindowShouldClose(window)){

        float speed=inCar?0.3:0.1;

        if(glfwGetKey(window,GLFW_KEY_W)){
            playerX+=sin(angle)*speed;
            playerZ-=cos(angle)*speed;
        }

        if(glfwGetKey(window,GLFW_KEY_S)){
            playerX-=sin(angle)*speed;
            playerZ+=cos(angle)*speed;
        }

        if(glfwGetKey(window,GLFW_KEY_A)) angle+=0.05;
        if(glfwGetKey(window,GLFW_KEY_D)) angle-=0.05;

        // enter car
        float dx=playerX-carX;
        float dz=playerZ-carZ;

        if(glfwGetKey(window,GLFW_KEY_E) && sqrt(dx*dx+dz*dz)<2){
            inCar=!inCar;
        }

        if(inCar){
            carX=playerX;
            carZ=playerZ;
        }

        // shoot
        if(glfwGetKey(window,GLFW_KEY_SPACE)){
            Bullet b;
            b.x=playerX;
            b.z=playerZ;
            b.dir=angle;
            bullets.push_back(b);
        }

        for(auto &b:bullets){
            b.x+=sin(b.dir)*0.5;
            b.z-=cos(b.dir)*0.5;
        }

        // npc random walk
        for(auto &n:npcs){
            n.x+=((rand()%3)-1)*0.02;
            n.z+=((rand()%3)-1)*0.02;
        }

        glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT);

        glLoadIdentity();

        glRotatef(angle*57,0,1,0);
        glTranslatef(-playerX,-2,-playerZ-10);

        // ground
        glBegin(GL_QUADS);
        glVertex3f(-200,0,-200);
        glVertex3f(200,0,-200);
        glVertex3f(200,0,200);
        glVertex3f(-200,0,200);
        glEnd();

        // buildings
        for(int x=-10;x<10;x++)
        for(int z=-10;z<10;z++)
        if(rand()%4==0)
            cube(x*8,4,z*8,2);

        // car
        cube(carX,1,carZ,1);

        // player
        if(!inCar)
            cube(playerX,1,playerZ,0.5);

        // npcs
        for(auto &n:npcs)
            cube(n.x,1,n.z,0.5);

        // bullets
        for(auto &b:bullets)
            cube(b.x,1,b.z,0.2);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwTerminate();
}
